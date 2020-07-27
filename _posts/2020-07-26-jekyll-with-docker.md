---
layout: post
title:  "JekyllをDockerと組み合わせて使う"
categories: jekyll docker
---
ここ8ヶ月ほど、いろいろな事情があってマトモに勉強を勧められていなかったのですが、このままではいけないと感じて勉強を再開することにしました。

それに伴って学んだ事をアウトプットする場が欲しいなと思って、性懲りもなくサイトを立ち上げました。

## Jekyllを使う
以前にサイトを立ち上げた時は、CMSを使ったり自作の静的サイト生成スクリプトを使ってみたりといろいろやっていましたが、結局のところ運営が厳しくなって更新が滞ったり壊れた状態のままになってしまいました。

そこで今回は静的サイト生成エンジンとして有名なJekyllを使ってサイト運営を行うことにしました。

## Dockerと組み合わせて使う
Jekyllはとても簡単に利用できるプロダクトなので`gem install bundler jekyll`すれば大概の環境ですぐ使いはじめられますが、それで運営していたのでは少し簡単すぎるかな？と思い、Dockerと組み合わせて使うことにしました。
軽い気持ちでDockerとの組み合わせを考えてみたものの、案外手間がかかったので最初のポストはDocker上でJekyllを使うための方法をまとめていこうと思います。

## Dockerfileを書く
公式のDockerImageもありますが、使い勝手がイマイチだなと思ったので自分でイメージを作ることにしました。
下記のような内容で準備したDockerfileを`docker build ./ -t jekyll`などとしてビルドします。

```docker
FROM ruby:2.7

RUN mkdir /projects && gem install bundler jekyll && bundle config set path 'vendor/bundle'
ENV APP_ROOT /projects
WORKDIR $APP_ROOT
```

## 新しくサイトを作る
Dockerfileが出来上がった時点で大体の作業が終わっているので、あとはサイトを作っていくだけです。
（`10nin.com`は今回作ったサイトなので、他のサイトを作る場合は個別に指定します）
`~/work`とかのディレクトリに移動してから

```console
docker run -ti -p 4000:4000 -v $PWD:/projects --rm jekyll jekyll new 10nin.com
```

で新しくサイトを作ります。

余談ですが、macのDockerで上記コマンドを打ち込むとものすごく時間がかかります。
コーヒーを淹れてじっくり楽しむぐらいの余裕を持って作業すると良いと思います。

さて`~/work`で作業した場合、`$PWD`は`~/work`として解決されるので、`~/work`ディレクトリ配下に今指定したサイト（この場合は`10nin.com`）が出来上がります。
この後以降は作成したプロジェクトフォルダをDockerの`-v`オプションでコンテナ側にマウントして作業していくので、サイトの構築作業はローカル側で行うことができます。

## Jekyllサーバを立ち上げる
サイトが出来上がったので、Jekyllの開発用サーバを起動して動作確認をします。
同じく`~/work`ディレクトリ配下などで、下記コマンドを実行します。
ここでデフォルト設定だと開発用サーバは`127.0.0.0`で起動してしまうので、`0.0.0.0`で起動するよう`--host`オプションを指定しています。（なお`$PWD/10nin.com`の`10nin.com`部分は、先ほど作ったサイトの名前です）

```console
docker run -ti -p 4000:4000 -v $PWD/10nin.com:/projects --rm jekyll bundle exec jekyll serve --host=0.0.0.0
```

コマンドが成功して開発用のサーバが起動すれば、`http://0.0.0.0:4000`で生成したサイトを確認することができます。

## サイトの設定を行う
サイトが確認できたら、諸々の設定を行います。
Jekyllのドキュメントにしたがって設定してくことになりますが、ローカルのJekyllで作業するのと異なり、Dockerで行った場合には（コンテナのタイムゾーンに依存しないようにしたいので）タイムゾーン指定を行っておくのが良いと思います。
なので`_config.yml`には`timezone: Asia/Tokyo`オプションを書き込むようにします。

JekyllとDockerを組み合わせて使う時にやらなければならない仕事はこれで全てです。

## サイトをビルドする
一通り設定やコンテンツ作りが終わったら、サイトをビルドしてやります。
```console
docker run -ti -p 4000:4000 -v $PWD/10nin.com:/projects --rm jekyll bundle exec jekyll b
```
あとはビルドしたサイトを公開するだけです。

## コマンドをスクリプトにまとめておく
Dockerを使ったコマンドはオプションも含めて長くなりがちなので、私はよく使うコマンドを次のようなスクリプトにまとめて使っています。

```bash
#!/bin/sh
if [ ${#} -eq 2 ];then
  CURRENT=`pwd`
  WORK_DIR="${CURRENT}/${2}"
  case ${1} in
    "s") docker run -ti -p 4000:4000 -v ${WORK_DIR}:/projects --rm jekyll bundle exec jekyll s --host=0.0.0.0;;
    "b") docker run -ti -p 4000:4000 -v ${WORK_DIR}:/projects --rm jekyll bundle exec jekyll b;;
    * ) echo "boot development mode `basename ${0}` s [SITE_NAME] or build your site `basename ${0}` b [SITE_NAME].";;
  esac
else
  echo "boot development mode `basename ${0}` s [SITE_NAME] or build your site `basename ${0}` b [SITE_NAME]."
fi
```

## まとめ
以上がJekyllとDockerを組み合わせて使う方法でした。

作業している間は、Dockerfileで`bundle config set path 'vendor/bundle'`してプロジェクトごとにgemファイルを持たせてやるようにしないとコンテナを維持する必要があるために苦労したり、マウントしている作業パスを勘違いしてしまってゴミファイルが作られてしまったりしました。
ただ、それらを除けば普通にJekyllを使ってサイト構築するのと変わらないので、運用の手間も大きく増加するわけではなく、比較的簡単に利用できるものだと思います。
