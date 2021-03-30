---
layout: post
title:  "Railsを使うときの細かいTips"
categories: ruby ror
---
Railsの教科書の類には登場しないが、実務上知っていると便利というTipsをまとめます。
雑多なメモの類なので、まとまりがあるものではありませんが、知っていると便利なものばかりだと思うので、ご存じなかった方はこの機会に活用してみてください。

### RSpecの実行時、最初に失敗したspecで止める

プロジェクトが大きくなると、RSpecの実行にやたらと時間がかかってしまいます。
そういう場合にはファイルを指定して実行したり、specの側でタグ付けして実行範囲を限定したりして手早く実行できるようにするものかと思います。
ただ、テストが失敗したらそこで止まってほしいとケースもよくあって、その場合にはこんなコマンドで要望を実現することができます。

`$ bundle exec rspec --fail-fast`

私はテストを書くためにDSLを使うという発想が嫌いで、その中でもRSpecはRubyを書く時からのコンテキストスイッチのコストが大きくて嫌いです。
業務上使われている場合は仕方ないものとして受け入れるしかありませんが、本当にこんな長大なテストフレームワークが必要なプロダクトなのかは立ち止まって考えたほうが良いと思う事がよくあります。

### 接続中のデータベースの情報を知る

- Rails consoleからデータベースへの接続情報を取得するためには、`ActiveRecord::Base.connection_config`を参照してやります。
- Rails consoleがDBに接続するために使っている生の情報を見ることができるので、接続情報を管理しているファイルにアクセスしなくても接続先の情報を簡単に参照することが可能です。

### Railsを通じてDBのコンソールにアクセスする

- Railsを使っているとDBのコンソールを立ち上げて直接DBの中身を見たい時があります。
- 多くの教科書では、そういう時はDBのクライアントプログラムを立ち上げて云々と書かれていますが、下記のコマンドを使うとRails経由でDBのコンソールコマンドを立ち上げられるので便利です。
- `$ rails dbconsole`

### Rails Consoleからテーブル定義を覗き見る

- 「このモデルに対応するテーブルの定義はどうなっているのか？」というのを参照するときに使います。
- 例えば Users テーブルであれば `User.column.map {|c| {c.name => c.sql_type_metadata}}`等としてやればカラムの情報を一覧で取得できます。
- 何らかの事情でRailsの作法に従っていないテーブル名を使っている場合には、`User.tabale_name`で接続先のテーブル名を取得できます

### Rails Console上から生のSQLを実行して結果をHashで得る

- Railsを使っていると、「DBへの問い合わせはモデル経由で行うべし」という一種の宗教じみた空気を感じることがありますが、生のSQLを投げたほうが数段手早く情報を取得できるケースも多々あります。
- 具体的には、Rails consoleで作業している最中にDB自体の情報をメタテーブルから参照したいような場合に知っていると役立つTipsです。
- 例えばMySQLであれば、`ActiveRecord::Base.connection.select_all("SELECT @@SESSION.sql_mode").to_hash`でセッションで有効になる`sql_mode`を得るような使い方ができます。

### まとめ

知っている人にはどうという事のないTipsばかりですが、「これは便利」と感じたものを紹介しました。
気が付いたら時々ネタを追加しようと思います。