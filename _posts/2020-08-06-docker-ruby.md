---
layout: post
title:  "DockerComposeでRuby on Rails6とMySQL8を組み合わせて動かす"
categories: docker ror mysql
---
ちょっとwebアプリの検証をしたくてDockerCompose上にRails6 x MySQL8環境を作ろうと思ったのですが、案外ハマりどころが多かったのでメモに残しておくことにしました。
途中、たびたび`pwd`で作業ディレクトリを確認していますが、どこで実行しているのか迷わないようにするための解説的な意味合いが強いので、理解できている人は無視してください。

## 作業ディレクトリの作成〜下準備

### 作業ディレクトリを作って入る
メインになるプロジェクトのディレクトリと、ソースコードを放り込むディレクトリを作ります。

```console
mkdir -p ~/docker-ruby/src
cd docker-ruby
touch Dockerfile
touch Gemfile
```

### Gemfileを書く
最低限の内容をもったGemfileを書きます。
rails入れるときに上書きされて書いている内容が壊れてしまうので、あとで使いまわせるようにDockefileと同じディレクトリにオリジナルを入れておきます。
また、Gemfileを配置したら、touchコマンドを使ってGemfile.lockだけ作っておきます。

```console
pwd #=> ~/docker-ruby
vi Gemfile
cp Gemfile src/Gemfile
touch src/Gemfile.lock
```

Gemfileの内容はこんな具合です。6系で固定しても良いと思いますが、メジャーバージョンが上がるのは年単位で先の話なので今は無視しています。

```Gemfile
source 'https://rubygems.org'
gem 'rails'
```

### Dockerfileを書く
せっかくなのでRuby 2.7を使うことにしました。
ハマりどころとして、Rails6からはWebpackerが標準で入ってくるのでyarnを整える必要があるため、その辺を追加するようにしています。
（もっと良いやり方があるかもしれないです）

```Dockerfile
FROM ruby:2.7

# シェルスクリプトとしてbashを利用
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# 必要なパッケージのインストール
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs

# yarnパッケージ管理ツールインストール
RUN export APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1 && apt-get update && apt-get install -y curl apt-transport-https wget && \
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt-get update && apt-get install -y yarn

# 作業ディレクトリの作成と設定
RUN mkdir /app
ENV APP_ROOT /app
WORKDIR $APP_ROOT

# ローカルで作ったGemfileをコンテナ内の作業ディレクトリに配置する
ADD ./src/Gemfile $APP_ROOT/Gemfile
ADD ./src/Gemfile.lock $APP_ROOT/Gemfile.lock

# Gemfileのbundle install
RUN bundle install
ADD src $APP_ROOT
```

### docker-compse.ymlを用意する
閑話休題。KubernetesとかRailsとかdoocker-composeとかを扱っていると一番多く読み書きするのがソースコードじゃなくてYAMLのファイルになりがちだと思いませんか？

さて、docker-compose.ymlファイルを作ります。

```console
pwd #=> ~/docker-ruby
vi docker-compose.yml
```

データベースにはMySQL8の公式イメージをそのまま使うことにしました。
また、`DB_USER`と`DB_PASS`という名称で環境変数をexportして、docker-compose.yml内に秘密にしておくべき情報を記録しないようにしています。（今回の例だとDB_USERはデータベース名としても使われるので、一旦root固定です）

```console
export DB_USER="root"
export DB_PASS="db_password"
```

二つ目のハマりポイントがここにあって、Docker + MySQL8の環境だと、[こちらのQiita記事](https://qiita.com/yensaki/items/9e453b7320ca2d0461c7)にあるように`caching_sha2_password`の認証形式に対応できないので、コマンドラインオプションでAuthentication Pluginを変更して運用するようにしています。

```yaml
version: '3'
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: ${DB_USER}
      MYSQL_ROOT_PASSWORD: ${DB_PASS}
    ports:
      - "3306:3306"
    command: --default-authentication-plugin=mysql_native_password
    restart: always

  app:
    build: .
    command: rails s -p 3000 -b '0.0.0.0'
    volumes:
      - ./src:/app
    environment:
      - DB_USER=${DB_USER}
      - DB_PASS=${DB_PASS}
    ports:
      - "3000:3000"
    links:
      - db
```

これで大体の準備が完了しました。

## 環境のビルド〜データベース設定の修正

### rails newして下地を作る
`rails new`します。このときにコンテナがビルドされるので、ちょっと時間がかかります。
どうでもいいですけど、`nokogiri`や`sassc`等のnative extensionの導入は、もっと高速になって欲しいと思っています。

```console
docker-compose run app rails new . --force --database=mysql --skip-bundle
```

ここで、newするときに`--skip-bundle`が付いていると

```
Could not find gem 'mysql2 (>= 0.4.4)' in any of the gem sources listed in your Gemfile.
Run `bundle install` to install missing gems.
```

などとエラーが出てしまいますが、一旦無視して先に進みます。

### database.ymlの編集
ビルドできたら、データベースの設定ファイルを編集します。
ちなみに、ファイルの編集コマンドに`vi`を使っていますが、実際はVSCodeを使って作業しています。

```console
pwd #=> ~/docker-ruby
vi src/config/database.yml
```

`docker-compose.yml`でDBのユーザとパスワードを環境変数として定義しているので、`database.yml`でもそれを使うようにします。
ということで、下記の要領で`default`部分の設定を書き換えます。
（`host`に`docker-compose.yml`で指定したデータベースコンテナの名前を与えるのを忘れないようにします。）

```yaml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV.fetch("DB_USER") { "root" } %>
  password: <%= ENV.fetch("DB_PASS") { "" } %>
  host: db
```

## コンテナビルドをしなおして、webpackerを導入する

設定も終わったのでコンテナをビルドしなおしてから、webpackerのインストールを行います。
（webpackerのインストールも結構時間がかかるので、気長に待ちます。）
最後のハマりポイントがここで、webpackerのインストールを忘れるとappコンテが側を起動しても、何かしようとした途端に落ちてしまいます。（単に`docker-compose up`しただけだと普通に動いているように見える(UP状態に見える）のがたちの悪い所です）

```console
pwd #=> ~/docker-ruby
docker-compose build
docker-compose run app rails webpacker:install
```

appコンテナのビルドに際して、下記のようにtzinfo-dataのエラーが出ますが、私と同じDocker for macを使っている環境ならば無視してしまって大丈夫だと思います。(エラーメッセージにある通り、x86-mingw32/x86-mswin32/x64-mingw32/javaの各環境で依存関係が生じるものです)

```console
The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x86-mswin32, x64-mingw32, java. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x86-mswin32 x64-mingw32 java`.
```

また、docker-composeコマンドはよく使うので、適当にエイリアスをつけておくと良いと思います。

## DBをつくって動作確認する
Railsを通じてDBを作ります。ここまで来ればもう難しいことはないです。

```console
pwd #=> ~/docker-ruby
docker-compose down # 一回すべてのコンテナを落とす
docker-compose up -d
docker-compose ps #=> コンテナがUpしていることを確認する
docker-compose exec app rails db:create
```

出来上がったら`http://0.0.0.0:3000`にアクセスして動作確認します。
Yay!ページが表示されたら動作確認完了です。
これでDockerComposeとRails6 x MySQL8を組み合わせた環境ができあがりました！

![Rails6のYay!ページ]({{absolute_url}}/images/rails6_yay.png)