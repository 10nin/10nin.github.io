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
form_with url: file_import_path, method: :post, multipart: true, remote: true do |f|
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
解決策として、先の回答にも上がってた remotipart という gem を導入し、問題解消を試みています。
