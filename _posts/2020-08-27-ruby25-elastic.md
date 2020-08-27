---
layout: post
title:  "ElasticAPMとRuby2.5を組み合わせるとSEGVで落ちる"
categories: ruby elasticapm
---
Ruby2.7への乗り換えの話ならばまだしも、この時期にRuby2.5の話を書くことになるとは思いませんでしたが、仕事でちょっとしたトラブルに遭遇したので書き残しておこうと思います。

### SEGVで落ちる

エラーとしては`./shared/bundle/ruby/2.5.0/gems/elastic-apm-3.8.0/lib/elastic_apm/transport/base.rb:83: [BUG] Segmentation fault at 0x000055de00000050` というような内容で、要するにElasticAPMのRubyクライアントがSEGVで落ちるという話です。

本家でも過去に何度か話題になっていてるようで、Ruby2.5系との組み合わせで発生するエラーのようです。（[Issue](https://github.com/elastic/apm-agent-ruby/issues/441)）

再現条件はよくわかっておらず、Ruby2.6系以降ならば発生しないのかも検証しきれていませんが、Ruby本家では対応するバグリポートは既に解決しているものとしてCloseされているので、再現条件がきちんとわかってくれば根本的な解決策を得られるのかもしれません。
（エラーから再現条件を見てRuby本体にパッチを投げられるとカッコイイですが、そこまでは至っていません）

### 対応策

私がぶち当たったのはunicornとの組み合わせでRailsのアプリを動かした場合だったので、unicornの設定ファイルの`after_fork`ブロックで`ElasticAPM.restart`を行って、プロセスがフォークされたらElasticAPMをリスタートさせる事で一応の解決を見ています。

これは設定によってElasticAPMをrestartさせるという運用回避策であって、根本的な解決を行うものではないですが、Webアプリ側でElasticAPMをとりあえず動かすことはできるようになりました。
