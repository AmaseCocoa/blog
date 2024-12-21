---
title: Misskeyのいろいろなものを別の言語で再実装してみてる話
description: media proxyとかsummaly proxyとか
published: 2024-12-21T14:25:00.000Z
draft: false
category: Misskey
tags:
  - Tech
---
> この記事は[Misskey Advent Calendar 2024](https://adventar.org/calendars/10208) 21日目の記事です
> 
> [← 20日目の記事](https://yumechi.jp/ja/blog/2024/misskey%E3%83%8F%E3%83%BC%E3%83%89%E3%83%8B%E3%83%B3%E3%82%B0/) | [22日目の記事 →](#)

<iframe src="https://adventar.org/calendars/10208/embed" width="620" height="362" frameborder="0" loading="lazy"></iframe>

[Media Proxy for Misskey](https://github.com/misskey-dev/media-proxy)や[summaly](https://github.com/misskey-dev/summaly)をいろんな言語で再実装してみてる話です。駄文ですが許してください

もともとはAPIラッパーの話をしようと思ってたんですが、色々問題が起こりすぎて間に合いそうになかったので変えました

## 何で作ったのか
~~特に理由はなかったりします~~ ただ作ってみたかったからっていうのが理由です (ちなみにsummaly-pyは初期は[summergo](https://github.com/nexryai/summergo)を無理矢理Pythonで動かすだけのお遊び的な実装でした)

ちなみにsummalyとMedia Proxyだとsummalyの方が実装が面倒でした

media-proxy-goはGoの勉強のために作ってみました (微妙なので使うのはおすすめしません)
## 発生した問題など
### summaly-py
~~最初から殆ど問題はなかったかもしれません~~ 強いて言えばcontent_length_limitがNone (null)な場合にエラーが返ってくるくらいです
### media-proxy
media-proxyは初期に使ってたライブラリのPillowだと大きな画像を処理できずにエラーを返してくる問題がありました。 ([#1](https://github.com/AmaseCocoa/media-proxy/issues/1))

結局本家で使われている[Sharp](https://sharp.pixelplumbing.com/)と同じlibvipsを使う[pyvips](https://github.com/libvips/pyvips)を使うようにしたら解決して少し早くなったような気がします。

GIFがおかしくなる[問題](https://misskey.io/notes/9ycexc1k071y0gdb)は結局GIFはそのまま返すようにして無理矢理解決しました。

### media-proxy-go
~~多すぎて覚えてない~~ 横長の画像をリサイズすると押し潰される
## 作ってみた感想
* summaly-pyは使ったことのないXPathを書いていて少しだけ大変だった
* media-proxyは最初は大きいメディアを処理できなかったり問題が多かった (改善済み)
  * GIFは未だにバグるので処理できない
* media-proxy-goはWindowsでは使えない

~~ゴミみたいな感想ですね~~
## 最後に
~~記事を書いてたら足の力がだんだん入らなくなってきました。助けてください~~

記事を書くのって大変ですね
## Repos
#### media-proxy
::github{repo="AmaseCocoa/media-proxy"}

#### media-proxy-go
::github{repo="AmaseCocoa/media-proxy-go"}

#### summaly-py
::github{repo="AmaseCocoa/summaly-py"}

