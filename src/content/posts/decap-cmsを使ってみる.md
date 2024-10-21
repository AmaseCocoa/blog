---
title: Decap CMSを使ってみる
description: 自作CMSをやめてDecap CMSを使ってみた。
published: 2024-09-13T19:13:00.000Z
draft: false
category: Tech
tags:
  - decap-cms
  - hugo
  - git
---
自作CMSをやめてDecap CMSを使ってみた。

## 他の案

### Wordpress (ヘッドレス)

へッドレスCMSとしてWordpressを使う案。実際、実装は完成していた。でも微妙だったのでやめた。

### microCMS

日本製のヘッドレスCMS。転送量の制限が気になるので結局やめた。

### Decap CMS

旧Netlify CMS。Gitを使える。Netlifyとの連携が強い (Cloudflare Pagesとかでも動かせなくはない)。**Gitを使える** (ここ大事)

## なんで選んだの

* データベースが不要 (Wordpressを使う場合はこれが課題だった)
* 無料 (お金がないんです😇)

### それ以外に使った技術

* Pagefind (検索用。ホームとかにある検索ボックスがそれ)
* Hugo (ビルド時間が短くなりそうっていう勝手な妄想から。あとは使ったことがあるから)

#### UI

* TailwindCSS
* PhotoSwipe
* Barba.js v2 + GSAP (非同期転移。読み込みのときに一度真っ白になるのが気になるので…)

テーマは自作しました。

## 構築してみる

以下のコマンドを実行してhugoのサイトを作成する。テーマは[hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack)を使う。

```
hugo new site decap-site

cd decap-site/themes
git clone https://github.com/CaiJimmy/hugo-theme-stack.git
cd ..
```

完了したら`hugo.toml`を開いて以下の項目を追加する。

```toml
theme = 'hugo-theme-stack'
```

次に`hugo serve`を実行して正常に起動したら完了。

### Decap CMSを導入する

導入は意外と簡単にできた。
まずはstaticディレクトリに移動する。

```
cd static
```

次にstaticディレクトリにadminディレクトリを作成してadminディレクトリに移動。

```
mkdir admin
cd admin
```

config.ymlを作成して以下をペーストする。 

※このブログで使っているものです

```yaml
locale: "ja"
local_backend: true

backend:
  name: git-gateway
  branch: main

collections:
  - name: "post"
    label: "Post"
    folder: "content/posts"
    create: true
    slug: "{{fields.filename}}"
    preview_path: "posts/{{slug}}"
    type: "post"
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - {label: "Type", name: "type", widget: "hidden", default: "post", hint: "ファイルの種類 (通常は変更する必要はありません)"}
      - {
          label: "File Name",
          name: "filename",
          widget: "datetime",
          date_format: "YYYY-MM-DD",
          time_format: false,
          format: "YYYY-MM-DD",
          hint: "作成されるファイル名と記事のURLで使用されます。新規作成時のみ有効な設定です。",
        }
      - {
          label: "Publish Date",
          name: "date",
          widget: "datetime",
          date_format: "YYYY-MM-DD",
          time_format: "HH:mm:ss+09:00",
          format: "YYYY-MM-DDTHH:mm:ss+09:00",
          hint: "公開日の設定です。未来の時刻では公開されません。",
        }
      - {
          label: "Draft",
          name: "draft",
          widget: "boolean",
          default: true,
          hint: "下書きとして保存するかどうかを設定します。",
        }
      - {
          label: "Tags",
          name: "tags",
          widget: "list",
          hint: "タグはカンマ区切りで指定します。",
        }
      - {label: "Category", name: "category", widget: "list", hint: "カテゴリーはカンマ区切りで指定します。",}
      - {label: "Author", name: "author", widget: "relation", collection: "authors", searchFields: ["name"], valueField: "name"}
      - { label: "Body", name: "body", widget: "markdown" }
  - name: "authors"
    label: "Authors"
    folder: "content/authors"
    create: true
    slug: "{{fields.slug}}"
    fields:
      - {label: "Slug", name: "slug", widget: "string", hint: "英字である必要があります。"}
      - {label: "Name", name: "name", widget: "string"}
      - {label: "Bio", name: "bio", widget: "text"}
      - {label: "Avatar", name: "avatar", widget: "string"}
      - label: "Links"
        name: "links"
        widget: "list"
        fields:
          - {label: "URL", name: "url", widget: "string"}
          - {label: "Icon", name: "icon", widget: "string"}
      - {label: "Content", name: "body", widget: "markdown"}

media_folder: "static/uploads"
public_folder: "/uploads"
```

次にindex.htmlを作成して以下のコードをペーストする

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <meta name="robots" content="noindex" />
        <title>Content Manager</title>
        <script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>
    </head>
    <body>
        <script src="https://unpkg.com/decap-cms@^3.0.0/dist/decap-cms.js"></script>
    </body>
</html>
```

`hugo serve`でサーバーを起動したら`サーバーのアドレス/admin`にアクセスして画像の画面が表示されるか確認する。表示されたらNetlifyの設定をする。

![](/uploads/decap.jpg "ログイン画面")

### Netlifyの設定

ここからはこのブログの管理画面を使います (?)

Netlifyのプロジェクトの作成に関しては割愛します。

![](/uploads/netlify-admin.jpg "※Builds are stoppedと表示されているのはこのウェブサイトはPagefindを使う都合で別の場所でビルドしているからです。")

サイドバーの「Integrations」をクリックし、出てきたページの「Identity」をクリックします。すると、右に「Netlify Identity」というのがあるのでそれの下にある「Enable」をクリックしてIdentityを有効化します。

![](/uploads/netlify-integrations.jpg)

次に「Site configuration」に戻って、下にスクロールしたところにあるIdentityをクリックします。すると画像のような画面が表示されるので、「Registration preferences」を「Open」から「Invite only」に変更します。

![](/uploads/identity.jpg)

次にintegrationsからidentityの部分に戻ります。Netlify Identityの下にあるviewをクリックして転移した画面で表示される、「Invite users」をクリックします。表示されたモーダルに自分のメールアドレスを入力したあと、sendをクリックします。

![](/uploads/identity-invite.jpg)

すると画像のようなメールが送信されているはずなので、それの**リンクをコピー**します。

※このままクリックしても管理画面ではない場合は登録されないため

![](/uploads/netlify-identity-invited.jpg)

\`#を除いたリンク/admin/#以降の文字列\`のように置き換えてアクセスしてください。

するとおそらく管理画面が表示されます。

次にNetlifyのGit-Gatewayを有効化します。

まず、Site configurationのIdentityに戻ります。下の方にスクロールして「Git Gateway」の項目を見つけたら、「Enable Git Gateway」をクリックしてGit Gatewayを有効化します。これで導入は終わりです。

## 感想

簡単にできた。

今度Netlify以外での方法も書くかもしれません。
### あとがき
これ投稿してからPhotoSwipeのCSSを同梱してないことに気づいた。
