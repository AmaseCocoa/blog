---
title: 何を狂ったのかCMSを一から作った話
description: 何故かCMSからブログを作ってしまいました。
image: https://media.misskeyusercontent.jp/io/e41a7d28-8783-4496-8f50-a39e0d8b6ae8.webp
published: 2024-08-20T16:04:00.000Z
draft: false
category: Tech
tags:
  - 自作CMS
---
何故かCMSからブログを作ってしまいました。
> ちなみに今は[Decap CMS](https://decapcms.org/) + [Fuwari (Astro)](https://github.com/saicaca/fuwari)の構成に移行しているので自作CMSは使ってません
## どうして？
なんとなく興味があったから。
![管理画面](https://i.imgur.com/iuUKckJ.jpeg)
*管理画面とされているページ*
## 使ってる技術
### バックエンド
* FastAPI
* uvicorn
* Python-Markdown (を少し拡張してます)
* PostgreSQL (CockroachDB)
* ちなみにスキーマいじればどのDBでも使えると思います
* Prisma側の検索機能も対応したいならPostgresかMySQLしか駄目かも
* Prisma
#### フロントエンド
* Bootstrap
* EasyMDE (WYSIWYGエディター)
#### サービス
* Render (PaaS)
* CockroachDB Cloud
## amsc.pages.devから変わった点
* 静的サイトではなくなった
* サーバー側でレンダリングしたページを返す形式になった
* Twemojiが使えるようになった
* ただし絵文字を直接変換することはできなくて絵文字のコード (:thumbsup:なら`:thumbsup:`)を打つ必要がある
* 将来的には絵文字を直接変換できるようにするか絵文字ピッカーを実装したい
* 動的にcssやjsを圧縮できるようになった
* 管理画面ができた
* 投稿が簡単に編集できるようになった
* 削除も同じく
* 閲覧数がカウントされるようになった
* 精度が低いので見れないようになってるけど一応カウントされてます
* 検索機能が実装された ← NEW!!!
## やりたいこと
* 画像自体をエディターから上げられるようにしたい
* 現状はできない
* 実現するならS3とかR2みたいなのを使う必要がある気がする
* MFMを描画できるようにしたい
## あとがき
結構速さは重視してたりします
お陰で評価は結構高いです
![PageSpeed Insights](https://lh3.googleusercontent.com/pw/AP1GczPodlshE4DdbiFSj94RhrdRBAGtI7Eu3M-uq_ml4LJoH3P8YE8RLAGQfwY0zYKo_arXrXm4xNQ4hv5OjHGAP0pCAqWShRYPMme-PF6x3J32fuKL3o1NpDxZTRo25tGHhcVrdTYRZnFo_Xbk9pXBcp_H=w1899-h916-s-no-gm?authuser=0)
*モバイル*
![PageSpeed Insights](https://lh3.googleusercontent.com/pw/AP1GczNYamuj-IQ0hPuWjs-u8YLQaYVdhQZSVDe7HBaEZJicNzII4NyduO_8n2AOpy9_ctYg6poBeK6GnJkkzkY56w4zCgYsLXZEc-DttmKL3alE0hB81RK1bBCys8_1LkkNv4d2fg1ONwc0SCTbWhtzfbHw=w1139-h524-s-no-gm?authuser=0)
*デスクトップ*
![](https://lh3.googleusercontent.com/pw/AP1GczOjZroI7oVb4Q6yQQVE0sNEXaIfFW_ncvq8hpik-68c50D2ukG6-N4aecumQdXuXFFeJ6WCGlzuzGeh8-HCQdbAT-AUt_xxzFXPl1yByd3jshuxuMv6uov9F4CjLl3F3ICQw5Rzq5wG4q6J9OJDTskZ=w1901-h911-s-no-gm?authuser=0)
*記事ページはモバイルだと微妙かも*
![](https://lh3.googleusercontent.com/pw/AP1GczOv1vFto-ky9Q6tg3NSZ7NiKFdYXlc0OHq7s27QSg68R_EWigM-2Co7igMWBWTd7U2dXCD394n42uyjiFA8ATOt9jr0i_6CR9Mdf_FAVmrNY07jqmcpX2GdipZ_9Ttp6ruZhpxDnIL5jQnjaF3LEKsI=w1046-h505-s-no-gm?authuser=0)
*記事ページでもデストップだといい感じ*
