---
layout: post
title:  "Ubuntuでcargo-editのインストール時にNo package 'openssl'エラーになった時の対処"
categories: rust cargo
---
新しいPCにRust環境を構築していて、cargoの便利ツールであるcargo-editの導入時に表題のエラーでつまずいたのでメモ。

Ubuntuで

`cargo install cargo-edit`

した時に、 `No package 'openssl'` エラーになったら

`sudo apt-get install libssl-dev`

することで解決できる。ちなみに、 `pkg-config` パッケージも必要なようなので、不足している場合は同時に `apt install` するのが良いと思います。
