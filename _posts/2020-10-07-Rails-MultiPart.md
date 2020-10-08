---
layout: post
title:  "ファイルアップロードを伴うフォームで、remote trueが効かない問題でハマった話"
categories: ruby ror
---
ある外部システムから出力したファイルをインポートする仕組みを作っていた中で、ハマった事象があったのでメモとして残します。

## 問題の概要

ファイルを取り込むときに、Ruby on Railsを用いて次のようなフォームを構築していました。
なお、対象のコードはRails 4時代から継ぎ足し継ぎ足し使われてきていて、現在は5.2で稼働しています。

```slim
form_with url: file_import_path, method: :post, multipart: true, data:{remote: true} do |f|
  ... なんかいろいろフォームの要素
  = file_field_tag :file
  = f.submit 'インポート', class: 'js-processing-btn'
.import-result
  | 処理結果を表示します
```

対応するControllerのActionはこんな具合です

```ruby
def file_import
  file_importer  = FileImporter.new(params[:file])
  file_importer.import!
  render json:{ errors: file_importer.errors }
end
```

要するにファイルアップロード機能を持ったフォームを開発したいワケです。
当初の目論見としては、「インポート」というボタンを押下すると非同期で通信が飛んでいって、サーバからJSONが返され、帰ってきたJSONの情報をフォーム下部に設けた処理結果の部分に表示するという寸法でした。
しかし、このコードは期待した通りに動きません。実際に「インポート」ボタンを押下すると、単にJSONがレンダリングされたページに遷移するだけの状態になってしまいます。

## 調べて試して、失敗

「Ajaxが思い通り動かない」というのは結構あるあるらしく、すぐに解決策が見つかるかな？と思いましたが、まったく思うように行きません。
試したことの一例を挙げると...

- Actionで、`respond_to { |format| format.js { rednder ...` と `respond_to` を使ってサーバ側のデータを書き出す
- `file_import.json.jbuilder` というviewを作って、それを介してJSONを書き出すようにする
- (継ぎ足し継ぎ足しのコードだったので)js側で使われていたjquery_ujsではなく、rails_ujsを読み込むようにする

いずれも全く効果無しでした。

## multipartが怪しい

1-2時間ぐらい試行錯誤しても解決しなかったので、先輩エンジニアに助けを求めたところ「過去に`multi-part`と`remote true`の合わせ技でハマった記憶がある」とのこと。
まさかmuti-partが悪さをしているとは思いもよらなかったので、早速これらをキーワードに調べてみると、すぐに答えに行き着きました。

### multipart: true だと、 remote: true を指定してもAjaxなフォームにならない

結論としては、[teratailに書かれていたこちらの情報](https://teratail.com/questions/153491)の通りなのですが、`mutipart:true`を指定したフォームではリクエストも必ずhtmlで飛んでいくため、Ajaxなフォームにならないという事がわかりました。
解決策として、先の回答にも上がってた remotipart という gem を導入し、問題解消を試みます。

### gemの導入と準備

ネット上の情報では、Rails 5系ではGithub情にある、特定のforkされた remotipart を使わないといけないという記述がありますが、既にこのパッチは本体に取り込まれているため、下記をGemfileに追加するだけでOKです。

```ruby
gem 'remotipart'
```

bundle install してgemをインストールします

```bash
bundle install
```

最後に、application.js等にremotipartを使う記述を加えます。
独自に書き加えているものもあると思いますので、jquery系の読み込みをしている箇所の最後に加えることにしました。

```js
//= require jquery.remotipart
```

これで準備は完了です

### 実際に使ってみる

実際に使うときは、formが確実に`remote:true`で送られる必要がありますので、form_withを使うとか(form_withやform_tagの引数に)`data:{remote:true}`を渡して意図した通り動くように調整しておきます。
私の場合は最初に使っていたフォームがそのまま使えたので、このままで行きます

```slim
form_with url: file_import_path, method: :post, multipart: true, data: {remote: true} do |f|
  ... なんかいろいろフォームの要素
  = file_field_tag :file
  = f.submit 'インポート', class: 'js-processing-btn'
.import-result
  | 処理結果を表示します
```

### コントローラ側の罠

コントローラ側はJSONをレンダリングして。。。と思って、当初の通り下記のコードを動かそうとしたのですが、ここで問題が。
このコードで実行すると `Uncaught SyntaxError: Unexpected token ':' ...` というエラーで怒られてしまいます。

```ruby
def file_import
  file_importer  = FileImporter.new(params[:file])
  file_importer.import!
  render json:{ errors: file_importer.errors }
end
```

### ビューを作る

調べたところjsonを返すのはNGで、`file_import.js.coffee`というビューを用意して、このビューの中にCoffeeScriptで処理を書いてやる必要がある事がわかりました。
私のプロジェクトでは古いコードを引き継いでいるのでCoffeeScriptでビューを用意しましたが、場合によってはjsのビューで事足りるのかもしれません。

なお、`file_import.json.jbuilder`というビューを作ってJSONを返す試みをしてみましたが、これも同様のエラーが出てしまってNGでしたので、JSONを返すのは諦めました。
このように、CoffeeScriptのビューで処理完了後の挙動を書く事で、Ajaxな挙動を実現する事ができました。

わかっている人にとってはなんということのない内容かもしれませんが、案外ネット上に使える情報がなかったので、ここにたどり着くまで苦労しました。
