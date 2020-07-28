---
layout: post
title:  "Jekyllで作ったサイトをGitHub Pagesで独自ドメインで公開する"
categories: jekyll github
---
勢いでJekyllを使ってサイトを立ち上げることにしたものの、どこでどう運用していくかは決めていませんでした。
今まで通りVPSを借りてサーバを立てて、その上でWebサーバを動かして……とも思いましたが、せっかく新しくやるので、少し違った趣向でやろうと思い直し、GitHub Pagesを使ってみることにしました。

## GitHub Pagesについて
GitHubには、特定の命名規則のリポジトリを作ると、そこをウェブページとして公開できる仕組みがあります。
これをGitHub Pagesと呼んでいて、フリープランであってもpublicなリポジトリならば利用することが可能です。

今回は最終的にウェブページとして全世界に公開する内容ですから、特に秘匿する情報もありません。
さらに、独自ドメインやSSLのサポートまでしてくれるとあって、新しいウェブサイトはGitHub Pages上で公開することに決めました。

ちなみに、似たサービスにGitLabのGitLab Pagesというものがありますが、特に比較検討はしていません。
GitHub Pagesがリポジトリそのものを公開するタイプのサービスであるのに対して、GitLab Pagesはビルド成果物を静的なWebサイトとして公開するものなので、ビルド時にちょっとした仕掛けを入れたい時なんかはGitLab Pagesの方が良いかもしれません。
（いずれにせよ独自ドメインでの公開なので、GitLab Pagesの方がよければそちらに乗り換えるかもしれません）

## 事前に調べたけどさして影響がなさそうだったことあれこれ
GitHubでサポートされている[Jekyllのバージョンが少し古い](https://pages.github.com/versions/)ので、Gemfileを書き換えてバージョンを落としておく必要があるかな？とも思いましたが、実際に4.1.1のJekyllを使ってサイト公開してみると大した影響はないように見えます。
3.xと4.x系での違いが出ているところを踏み抜かなければ、これといった対応は必要なさそうだと判断しました。


## 公開用のリポジトリを作って公開する
GitHub Pagesの使い方については、[公式に素晴らしいドキュメントが用意されている](https://docs.github.com/ja/github/working-with-github-pages/about-github-pages)のでこれをなぞっていく形で進めていきます。

まずはGitHub側に公開用のリポジトリを作ります。
GitHub Pagesに使える公開用のリポジトリ名は`<username>.github.io`の決まりがありますので、`10nin.github.io`でリポジトリを準備し、public設定で作成しました。
リポジトリを作り終えたら、 下記の要領で手元の作業ディレクトリをgitに対応させ、リモートのリポジトリをoriginに設定してからpushすれば、早くも [http://10nin.github.io](http://10nin.github.io) のURLでサイトが公開されます。
（今回はmasterブランチをそのまま公開しています）

```console
cd ~/works/10nin.com
git init .
git add -A
git commit -n -m 'first commit'
git remote add origin git@github.com:10nin/10nin.github.io.git
```

▼記事の用意等を終えたらpushします
```console
git push --set-upstream origin master
```

## 独自ドメイン設定を行う
サイトを公開したら、リポジトリのSettingsの画面下部に向かって行って、Custom domain部分にカスタムドメインを設定します。
今回はサブドメインを使った`www.10nin.com`でサイトを公開したいので、これを入力してSaveボタンを押下し、情報を保存します。
（情報を保存すると、10nin.github.ioリポジトリ内にCNAMEファイルが作られます）

これでGitHub側の設定は完了したので、使っているドメイン管理会社のポータル等からDNS設定にCNAME設定を追加します。
例えば、私が使っているサービスでは下記のように設定を行いました。

![CNAME設定](/images/cname.png)