---
theme: gaia
_class: lead
paginate: true
headingDivider: 1
marp: true
---

# Figma API + Cloudflare + hono で<br>画像共有

ToKyoto.js #02
2024/01/12
@hush_in

# 自己紹介

![bg contain left:20%](https://try-figma-cloudflare.hushin.workers.dev/c1a9eb84-3d99-4991-a08c-210bfe85a021.png)

- はっしゅ (Ryo KUROIWA)
- 長野 → 京都 → 埼玉
- 前職で 10 年ほど e コマースのフロントエンド開発
- 株式会社エス・エム・エス
- 最近個人サイト [zatsuni.dev](https://zatsuni.dev/) のドメイン取った
  - Cloudflare の設定の簡単さに感動して放置

X: [@hush_in](https://github.com/hushin), Github: [hushin](https://github.com/hushin)

# Figma

- https://figma.com/
- 言わずと知れたデザインツール
- 無料で使える
- ホワイトボードの FigJam
  - ラフな図を描くのに便利

![bg right](https://try-figma-cloudflare.hushin.workers.dev/19511fb0-e380-43b7-b5fe-53a780706c41.png)

---

## アイデア

Figma/FigJam で書いた図をブログ等に埋め込みたい

Share から フレーム単位で iframe のコードをコピーできる

<iframe style="border: 1px solid rgba(0, 0, 0, 0.1);" width="800" height="350" src="https://www.figma.com/embed?embed_host=share&url=https%3A%2F%2Fwww.figma.com%2Ffile%2FgeJ0w0WA0LtyW6vxaaDnQK%2FGoogle-Material-Design%3Ftype%3Ddesign%26node-id%3D120%253A3%26mode%3Ddesign%26t%3DnDLSEYeaTlt45wA2-1" allowfullscreen></iframe>

---

## iframe のメリデメ

- メリット
  - 公式が提供してる方法なので簡単
  - 編集した内容がすぐに反映される
  - 拡大しても綺麗
- デメリット
  - Figma ファイルに誰でもアクセス可にしておく必要あり
  - 画像のほうがなにかと取り回ししやすい
    - ブラウザ以外での表示
    - レンダリングが早い

---

## そこで [Figma REST API](https://www.figma.com/developers/api)

https://www.figma.com/developers/api#get-images-endpoint

> GET image
> Renders images from a file.

```
$ curl -H 'X-FIGMA-TOKEN: figd_xxx' \
'https://api.figma.com/v1/images/geJ0w0WA0LtyW6vxaaDnQK?ids=120-3'

{
  "err": null,
  "images": {
    "120:3": "https://figma-alpha-api.s3.us-west-2.amazonaws.com/images/5161d8e2-a8e1-4334-9a3f-b7c7bc6258dd"
  }
}
```

[生成された画像](https://figma-alpha-api.s3.us-west-2.amazonaws.com/images/5161d8e2-a8e1-4334-9a3f-b7c7bc6258dd)

---

しかし

> The image assets will expire after 30 days.

30 日で期限が切れる

そこで 自分が以前から気になっていた Cloudflare Workers と組み合わせてみることに

# [Cloudflare Workers](https://www.cloudflare.com/ja-jp/developer-platform/workers/)

- サーバーレスアプリケーションの構築ができる
- JavaScript が動く
- 無料枠あり

# [Hono](https://hono.dev/)

フレームワークは人気がある Hono を使ってみた

```TypeScript
import { Hono } from 'hono';

const app = new Hono();

app.get('/', (c) => {
  return c.text('Hello Hono!');
});
```

# Demo

https://try-figma-cloudflare.hushin.workers.dev/my/

(Basic 制限あり)

- Basic 認証
- 画像一覧
- アップロード
- 元の Figma URL に飛ぶ

src https://github.com/hushin-sandbox/try-figma-cloudflare

---

## アップロード

![](https://try-figma-cloudflare.hushin.workers.dev/866e8cca-c8fa-413d-bb48-f2c5c6a6bc65.png)

---

## ダウンロード

![](https://try-figma-cloudflare.hushin.workers.dev/acd1e1d5-96d2-413a-be3f-6e356306a646.png)

参考

- [Cloudflare R2 もいいぞ！ - ゆーすけべー日記](https://yusukebe.com/posts/2022/r2-beta/)
- https://github.com/yusukebe/r2-image-worker

---

## 管理ページ

- https://hono.dev/guides/jsx を使って記述
  - 基本的に SSR
- クライアント処理は [htmx](https://htmx.org/) と script タグ直書き
- 参考 [Hono + htmx + Cloudflare は新しいスタック](https://zenn.dev/yusukebe/articles/e8ff26c8507799)

# 開発はまりポイント

- Node.js すべての機能が使えるわけではない
  - [一部の API のみ対応](https://developers.cloudflare.com/workers/runtime-apis/nodejs/)
  - [figma-api - npm](https://www.npmjs.com/package/figma-api) を使おうとしたが内部で axios を使っていて失敗
- JSX は使えるが SSR されるのでクライアントでリッチなことしようとすると大変
  - [htmx](https://htmx.org/) 使いこなせたらもっと簡単に書ける？
  - 複雑だったら Cloudflare Workers は API 利用に留め Web アプリは別にするのがいいと感じた
- サーバサイドの設計自信ない。個人制作なので雑でもいいかの精神

# 振り返り

- このスライドの画像は今回作ったものを埋め込んでいます
- もともと同じ URL で画像が再編集可能だと便利そうと思って作ったが、キャッシュと相性悪くて断念
- 勉強になったのでヨシ

# 感想

- 今までフロントエンド開発が中心で、<br>バックエンドあまり書いてこなかったが…
  - 馴染んだ TypeScript で書けるので楽しい！
  - 無料でいろいろ使えてサーバレスなのが嬉しい
- Wrangler(Cloudflare Workers の開発環境) と Hono の開発者体験が良い
