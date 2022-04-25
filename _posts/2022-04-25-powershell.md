---
layout: post
title: "手元のGitブランチを整理するPowerShellスクリプトを書いてみた"
categories: git powershell
---
仕事で開発をしていると、リモートでマージされたブランチがいつまでも手元の環境に残ってしまう事があり、不便に感じていました。

そこで、まるっと削除を行う次のようなPowerShellスクリプトをこさえました。

```powershell
git branch -v | Select-String "\[gone\]" | % {$_ -split " +"} | Select-String "feature|fix|hotfix" | % {git branch -D $_}
```

要するに `[gone]` のブランチのうち、 `feature`, `fix`, `hotfix` を含んでいるブランチを `git branch -D` で消すという操作です。

Linux環境でも似たようなスクリプトを使っていたのですが、Windowsだとgrepすら無くて調べるのに時間を要してしまいました。

今の所結構快適に動いてくれていて、手元のブランチもスッキリして良い感じです。

次は、ブログのエンジンをHugoに移そうと考えているので、そのあたりの作業ができたらまとめたいと思っています。
