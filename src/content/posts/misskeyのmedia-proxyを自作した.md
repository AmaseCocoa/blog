---
title: MisskeyのMedia Proxyを自作した
description: Media-Proxyを作成したのでそのコードについて説明したりするだけ
published: 2024-08-28T03:47:00.000Z
draft: false
category: Misskey
tags:
  - Misskey
  - Python
---
[Media-Proxy](https://github.com/AmaseCocoa/media-proxy)を作成したのでそのコードについて説明したりするだけ。

※これは古いバージョンのコードです。Pillowを使っているので大きな画像は処理できません
```python
import os
import logging
import traceback

import aiofiles
import aiohttp
import aiohttp.web as web
from aiohttp_cache import (
    setup_cache,
    cache,
)
from PIL import Image
import io
import urllib.parse

logger = logging.getLogger(__name__)


async def fetch_image(session: aiohttp.ClientSession, url):
    async with session.get(url) as response:
        if not response.ok:
            return None
        else:
            content_type = response.headers.get("Content-Type", "").lower()
            data = bytearray()
            while True:
                chunk = await response.content.read(int(os.environ.get("CHUNK_SIZE", 1048576)))
                if not chunk:
                    break
                data.extend(chunk)
            return data, content_type


@cache(expires=os.environ.get("EXPIRES", 86400) * 1000)
async def proxy_image(request):
    query_params = request.rel_url.query
    url = query_params.get("url")
    fallback = "fallback" in query_params
    emoji = "emoji" in query_params
    avatar = "avatar" in query_params
    static = "static" in query_params
    preview = "preview" in query_params
    badge = "badge" in query_params

    try:
        if not url:
            return web.Response(status=400, text="Missing 'url' parameter")

        try:
            url = urllib.parse.unquote(url)
        except Exception as e:
            return web.Response(status=400, text="Invalid 'url' parameter")

        async with aiohttp.ClientSession() as session:
            image_data, content_type = await fetch_image(session, url)

            if image_data is None:
                if fallback:
                    headers = {
                        "Cache-Control": "max-age=300",
                        "Content-Type": "image/webp",
                        "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                        "Content-Disposition": "inline; filename=image.webp",
                    }
                    async with aiofiles.open("./assets/fallback.webp", "rb") as f:
                        return web.Response(
                            status=200, body=await f.read(), headers=headers
                        )
                return web.Response(status=404, text="Image not found")
            if "image" not in content_type:
                logger.info("Media is Not Image. Redirecting to Response...")
                headers = {
                    "Cache-Control": "max-age=31536000, immutable",
                    "Content-Type": content_type,
                    "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                    "Content-Disposition": "inline; filename=image.webp",
                }
                return web.Response(status=200, body=image_data, headers=headers)

            image = Image.open(io.BytesIO(image_data))

            if emoji:
                image.thumbnail((128, 128))
            elif avatar:
                image.thumbnail((320, 320))
            elif preview:
                image.thumbnail((200, 200))
            elif badge:
                image = image.convert("RGBA")
                image = image.resize((96, 96))

            output = io.BytesIO()
            image_format = "WEBP" if not badge else "PNG"
            if image_format == "PNG":
                image.save(output, format=image_format, optimize=True)
            elif image_format == "WEBP":
                image.save(output, format=image_format, quality=80)
            output.seek(0)

            headers = {
                "Cache-Control": "max-age=31536000, immutable"
                if image_data
                else "max-age=300",
                "Content-Type": f"image/{image_format.lower()}",
                "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                "Content-Disposition": f"inline; filename=image.{image_format.lower()}",
            }

            return web.Response(body=output.read(), headers=headers)
    except Exception as e:
        print(traceback.format_exc())
        if fallback:
            headers = {
                "Cache-Control": "max-age=300",
                "Content-Type": "image/webp",
                "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                "Content-Disposition": "inline; filename=image.webp",
            }
            async with aiofiles.open("./assets/fallback.webp", "rb") as f:
                return web.Response(
                    status=200, body=await f.read(), headers=headers
                )
        return web.Response(status=404, text="Image not found")


app = web.Application()
setup_cache(app)
app.router.add_get("/proxy/{filename}", proxy_image)
app.router.add_get("/", proxy_image)
app.router.add_get("/{filename}", proxy_image)

if __name__ == "__main__":
    web.run_app(
        app, port=os.environ.get("PORT", 3003), host=os.environ.get("HOST", "0.0.0.0")
    )
```
#### モジュールの読み込み

```python
import os
import logging
import traceback

import aiofiles
import aiohttp
import aiohttp.web as web
from aiohttp_cache import (
    setup_cache,
    cache,
)
from PIL import Image
import io
import urllib.parse
```

- `aiohttp`と`aiohttp.web`：HTTPクライアントとサーバー。
- `aiofiles`：個人的にwith openだと気になるので。使う必要はあまりないかも。
- `PIL`（Pillow）：画像処理ライブラリ。圧縮などに利用します
- `urllib.parse`：URLのパースとエンコード/デコード用
- `aiohttp_cache`：キャッシュ

#### 2. ログの設定

```python
logger = logging.getLogger(__name__)
```

#### 3. 画像の取得

```python
async def fetch_image(session: aiohttp.ClientSession, url):
    async with session.get(url) as response:
        if not response.ok:
            return None
        else:
            content_type = response.headers.get("Content-Type", "").lower()
            data = bytearray()
            while True:
                chunk = await response.content.read(int(os.environ.get("CHUNK_SIZE", 1048576)))
                if not chunk:
                    break
                data.extend(chunk)
            return data, content_type
```

指定されたURLから画像を非同期で取得し、バイトデータとコンテンツタイプを返す。一気に取得するのではなく (一気に取得してしまうと大きなファイルでは遅くなるので)1MBづつチャンクで取得するようになっています。

#### 4. ルート部分

```python
@cache(expires=os.environ.get("EXPIRES", 86400) * 1000)
async def proxy_image(request):
    query_params = request.rel_url.query
    url = query_params.get("url")
    fallback = "fallback" in query_params
    emoji = "emoji" in query_params
    avatar = "avatar" in query_params
    static = "static" in query_params
    preview = "preview" in query_params
    badge = "badge" in query_params

    try:
        if not url:
            return web.Response(status=400, text="Missing 'url' parameter")

        try:
            url = urllib.parse.unquote(url)
        except Exception as e:
            return web.Response(status=400, text="Invalid 'url' parameter")

        async with aiohttp.ClientSession() as session:
            image_data, content_type = await fetch_image(session, url)

            if image_data is None:
                if fallback:
                    headers = {
                        "Cache-Control": "max-age=300",
                        "Content-Type": "image/webp",
                        "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                        "Content-Disposition": "inline; filename=image.webp",
                    }
                    async with aiofiles.open("./assets/fallback.webp", "rb") as f:
                        return web.Response(
                            status=200, body=await f.read(), headers=headers
                        )
                return web.Response(status=404, text="Image not found")
            if "image" not in content_type:
                logger.info("Media is Not Image. Redirecting to Response...")
                headers = {
                    "Cache-Control": "max-age=31536000, immutable",
                    "Content-Type": content_type,
                    "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                    "Content-Disposition": "inline; filename=image.webp",
                }
                return web.Response(status=200, body=image_data, headers=headers)

            image = Image.open(io.BytesIO(image_data))

            if emoji:
                image.thumbnail((128, 128))
            elif avatar:
                image.thumbnail((320, 320))
            elif preview:
                image.thumbnail((200, 200))
            elif badge:
                image = image.convert("RGBA")
                image = image.resize((96, 96))

            output = io.BytesIO()
            image_format = "WEBP" if not badge else "PNG"
            if image_format == "PNG":
                image.save(output, format=image_format, optimize=True)
            elif image_format == "WEBP":
                image.save(output, format=image_format, quality=80)
            output.seek(0)

            headers = {
                "Cache-Control": "max-age=31536000, immutable"
                if image_data
                else "max-age=300",
                "Content-Type": f"image/{image_format.lower()}",
                "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                "Content-Disposition": f"inline; filename=image.{image_format.lower()}",
            }

            return web.Response(body=output.read(), headers=headers)
    except Exception as e:
        print(traceback.format_exc())
        if fallback:
            headers = {
                "Cache-Control": "max-age=300",
                "Content-Type": "image/webp",
                "Content-Security-Policy": "default-src 'none'; img-src 'self'; media-src 'self'; style-src 'unsafe-inline'",
                "Content-Disposition": "inline; filename=image.webp",
            }
            async with aiofiles.open("./assets/fallback.webp", "rb") as f:
                return web.Response(
                    status=200, body=await f.read(), headers=headers
                )
        return web.Response(status=404, text="Image not found")
```

クエリパラメータを取得し、画像のURLを取得。

画像が見つからない場合やエラーが発生した場合の処理。

画像の種類に応じてサムネイルなどの処理を行い、適切なヘッダーを設定してレスポンスを返す。

#### 5. サーバーの設定

```python
app = web.Application()
setup_cache(app)
app.router.add_get("/proxy/{filename}", proxy_image)
app.router.add_get("/", proxy_image)
app.router.add_get("/{filename}", proxy_image)

if __name__ == "__main__":
    web.run_app(
        app, port=os.environ.get("PORT", 3003), host=os.environ.get("HOST", "0.0.0.0")
    )
```

`aiohttp`を使ってWebアプリケーションを設定。

キャッシュの設定。

`/proxy/{filename}`、`/`、`/{filename}`のルートに`proxy_image`ハンドラを設定。

アプリケーションを指定されたホストとポートで実行。ここは環境変数で変更できます。

~~ちなみに本当はintに変換しないといけないのにこのときのぼくは忘れていました~~
