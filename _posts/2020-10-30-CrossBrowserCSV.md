---
layout: post
title:  "Windows環境でCSVをアップロードすると、環境によってMIME-TYPEが変わる"
categories: crossplatform windows csv
---
あるフォーマットにのっとったCSVファイルをアップロードしてそれを処理し、データベースに取り込む。
いわゆるインポート機能を作っている時に表題の問題にぶち当たり、原因究明に時間を要したので備忘録として残します。

## CSVファイルかどうかを判定するためのコード
アップロードされたファイルが処理可能なCSVであるかを判定するために、下記のような判定コードを埋め込んでいました。

```ruby
def csv_file?
  @file.original_filename.end_with?('.csv') && @file.content_type == 'text/csv'
end
```

開発と動作確認はmacosで行い、ChromeとSafariのクロスブラウザチェックを実施したので大丈夫だろうと踏んでいました。

## Windows環境でファイルの処理に失敗する！
リリース前にクロスプラットフォームでのチェックを行うためWindows環境で動作確認をしたところ、うまく動かない事象が発生しました。
エクスポートしたファイルの問題なのか、データの問題なのか、はたまた環境の問題なのか。
いくつかの要因が重なっていてどこに問題があるのか判断がつきづらい状況でしたが、ログを埋め込む等して上記のCSV判定関数のところで落ちてしまうらしい事がわかりました。

## Windowsが付与してくるMIME-TYPEとそれに対応する改修
調査を進めたところ、どうやら`@file.content_type == 'text/csv'`でMIME-TYPEがCSVであるかどうかを調べている式が、Windows環境でだけfalseを返していることがわかりました。
しかも、Windows環境ではExcelのインストール状態によって返されるMIME-TYPEが違うことも判明したのです。

整理すると、次のような状態になることがわかりました。

- Windows(Excelなし) : `application/octet-stream`
- Windows(Excelあり) : `application/vnd.ms-excel`
- macosやLinux : `text/csv`

これらを考慮する必要があったため、最終的にCSVの判定関数を次のように書き直しました。

```ruby
MIME_TYPES = %w|application/octet-stream application/vnd.ms-excel text/csv|

def csv_file?
  @file.original_filename.end_with?('.csv') && MIME_TYPES.include?(@file.content_type)
end
```

これでWindowsからアップロードされるファイルも扱えるようになりました。
`text/csv`が標準だと思っていたのですが、Windows環境ではブラウザに関わらず上記のような挙動を示すので、このような対処が必要なのだと思われます。
