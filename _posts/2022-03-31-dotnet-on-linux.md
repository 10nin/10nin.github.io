---
layout: post
title: "WSL上のUbuntu環境に.NETの環境を整える"
categories: linux dotnet csharp
---
LTSもぼちぼち次のバージョンが出てくるので、今更感のあるメモですが。

## 署名キーとパッケージリポジトリの追加

次のコマンドでMicrosoftの署名キーを信頼し、dotnetの関連パッケージをAPT経由で取得できるようにパッケージリポジトリの追加を行う。

```bash
cd /tmp # 適当な作業フォルダで作業すると良い
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb # パッケージ取得
sudo dpkg -i packages-microsoft-prod.deb # インストール
rm packages-microsoft-prod.deb # 後始末
```

## .NET6環境のインストール

記事執筆時点では.NET6がLTSで利用できるので、次のコマンドで.NET6の開発環境をインストールする。

開発についてはSDKがあれば十分。

もしランタイムが必要な場合は、記事末尾の参考リンクにある情報に従って、ASP.NET Coreサポートのあるなしで、自身の環境に適した方をインストールすると良い。

```bash
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```

インストールが完了したら、インストールされた.NETのバージョンを確認しておく。

```bash
dotnet --version
```

## 参考リンク
[Ubuntuに.NETをインストールする](https://docs.microsoft.com/ja-jp/dotnet/core/install/linux-ubuntu)