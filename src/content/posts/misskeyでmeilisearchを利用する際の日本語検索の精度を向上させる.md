---
title: MisskeyでMeilisearchを利用する際の日本語検索の精度を向上させる
description: 体感ではそんな印象はないけど気になるので入れ替えてみる。
published: 2024-08-29T15:22:00.000Z
draft: false
category: misskey
tags:
  - misskey
  - meilisearch
---
体感ではそんな印象はないけど気になるので入れ替えてみる。
### Docker
アップデート手順はほとんど参考記事のものです。

コンテナを停止する。
```bash {linenos=true}
$ docker compose down
```

dumpを作成する。
```bash
$ curl  -X POST 'http://meilisearchのFQDN/dumps' -H 'Authorization: Bearer (MEILI_MASTER_KEYに設定した値)'
```
以下を実行してstatusがsucceededになったらok。
```bash
$ curl -X POST 'http://meilisearchのFQDN/tasks/(dumpsの返り値のtaskUid)' -H 'Authorization: Bearer (MEILI_MASTER_KEYに設定した値)'
```
`(composeに設定したmeilisearchのディレクトリ)/dumps`にdumpファイルが生成されているはず。

(念の為) 先に`(composeに設定したmeilisearchのディレクトリ)/data.ms`を別の場所にコピーしておく。

Meilisearchのdocker imageをMeilisearch 1.3.0系の
```yaml
image: getmeili/meilisearch:prototype-japanese-4
```
に置き換える。

次にこのコマンドを実行してdumpを取り込む
```bash
$ docker run -it --rm -p 7700:7700 -v /meili_data:/meili_data getmeili/meilisearch:prototype-japanese-4 meilisearch --import-dump /meili_data/dumps/(dumpの名前)
```

最後にコンテナを起動して完了。
```bash
$ docker compose up -d
```

#### 参考
- [Meilisearchのバージョンアップ手順(Docker版)](https://qiita.com/inunekousapon/items/0c7210c44c2b023b50d3)
- https://github.com/meilisearch/meilisearch/pull/3882
### 非Docker
> 自動でビルドされたバイナリをリリースに投げるようにしました。ビルドが面倒だったりリソースが不足したりする人はここからダウンロードすることもできます。
> ::github{repo="AmaseCocoa/Meilisearch-ForceJP"}
#### ビルドする場合
Dockerを使っていない場合は少し手順が複雑になる。

GitとRustがインストールされていない場合はインストールする。

Meilisearchのリポジトリをクローン、ディレクトリを移動する。
```bash
$ git clone https://github.com/meilisearch/meilisearch
$ cd meilisearch
```
v1.3.4をチェックアウトする
```bash
$ git checkout v1.3.4
```
最後にrustをアップデートして、中国語のトークン化を無効にしたバージョンのMeilisearchをビルドする。
```bash
$ rustup update

$ cargo build --release -p meilisearch -p meilitool --no-default-features --features "analytics mini-dashboard japanese"
```

最後に[ドキュメントの手順](https://www.meilisearch.com/docs/learn/update_and_migration/updating)を実行してMeilisearchを更新すれば完了。
#### 参考
- https://github.com/meilisearch/meilisearch/issues/4561#issuecomment-2058594295
- https://github.com/meilisearch/meilisearch/pull/3882
### 注意点
中国語のトークン化を無効にして精度を向上させているらしいので(このあたりは詳しくないのでわかりませんが) 多分中国語の精度がめちゃくちゃ落ちます。
