---
layout: page
title: F#を学ぶ:教科書の選定と学習していて困った事
disable_navi: "true"
---

## F#は.NET環境で動くOCamlだ

という訳で、F#を学ぶための教科書を探そうと思ったのですが、日本語で読める書籍はF#のブームがあった時に出版された古いものばかりで、調べてみれば調べてみるほど、最近の変更についていくには自分でキャッチアップする必要があることが分かってきました。

そこで、原点に立ち返ってまずはOCamlの.NET実装としてF#を学んでみることにしました。

さっそく、OCamlを学んだ時に買って、一通り勉強した後本棚の奥にしまい込んでいた、浅井 健一さんの「プログラミングの基礎」を引っ張り出してきました。

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=10nin-22&language=ja_JP&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4781911609&linkId=e8834437f66113d3965b531c662c9bf4" />

まずはこの教科書に沿って学んでいくことにします。

## 作業する場所を作る

勉強していくうえで、最初の方はREPL上でパチパチコードを打ち込んで評価する流れで問題なく学習を進められますが、ある程度のレベルまで進むとガシガシ書き換えられるソースコードがあった方が良いことに気づきます。

教科書でも4章の後半でファイルに分けてコードを書いていくように指示があり、書き換えられるソースコードファイルの必要性が高まってきます。

F#でこの手のファイルを用意するには、dotnetコマンドを使ってコンソールプログラム向けのプロジェクトを用意するのが手軽で良いと思いました。

具体的には`dotnet new console -lang="F#" -o ${プロジェクト名}`でファイルを用意します。
プロジェクト名の所は、`hello_world`とか`tutorial`とか、自分で分かりやすい名前を付けておけば、その名前でプロジェクトのディレクトリを作って中にひな形のプログラムを配置してくれます。

例えば、`dotnet new console -lang="F#" -o tutorial`で作成したプロジェクトはこんな構造で作成されます。

```
$ tree
.
├── Program.fs
├── obj
│   ├── project.assets.json
│   ├── project.nuget.cache
│   ├── tutorial.fsproj.nuget.dgspec.json
│   ├── tutorial.fsproj.nuget.g.props
│   └── tutorial.fsproj.nuget.g.targets
└── tutorial.fsproj
```
この中の、`Program.fs`ファイルにガシガシとプログラムを書いてはREPLに読み込ませて学習を進めていきました。

## 教科書を片手に進める上で気づいたこと

一通り学び終えて振り返ると、いくつかつまづいたり、気づいたりした所があったので列挙していきます。

- 命名規則
  - 教科書ではsnake_caseの命名規則が用いられていますが、Ionideのlinterによれば、F#ではcamelCaseを使うのが通例のようです。

- `#use`ディレクティブが無い
  - `dotnet fsi`コマンドで起動するF#のREPLには、`#use`ディレクティブが無いのでファイルに書いたプログラムを気軽に読み込むことができません。
  - `#load`ディレクティブが代替になるかと思いきや、読み込んだファイルが評価されなかったりしてイマイチ挙動が違っています。
  - 結局、起動時に`dotnet fsi --use:Program.fs`の要領で読み込ませたいF#のソースコードを記述したファイルを指定することで解決できました。

- 識別子に日本語を使うことができる
  - F#は.NETの上に載っている言語なので、変数や関数の識別子に日本語を使うことができます。手元で試してみた限り、絵文字はうまく処理できないようですが、ひらがな、カタカナ、漢字は使えるようでした。

- 識別子の大文字/小文字の区別がOCamlのそれに従わない
  - 小文字で始めると変数名であるとか、大文字で始めると型の識別子であるとか、F#では今回利用しているOCamlの教科書に書かれている命名のルールとは一致せず、.NETの命名ルールに従う事になるので、このあたりのルールについては習得しなおす必要があります。

- 思った以上にOCaml
  - より深く学んで行くとF#ならではの良さとか.NETの上に構築されていることによる恩恵が見えてくるのかもしれませんが、プログラミングの基礎を一通り舐めたぐらいの今のF#力では、ほとんどOCamlとの差が分からず、OCamlで学んだノウハウがそのまま開発に活かせるのは素晴らしいと感じています。
