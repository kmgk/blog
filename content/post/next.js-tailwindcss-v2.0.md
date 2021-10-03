---
title: 'Next.jsにtailwindcss v2.0を導入する'
date: 2020-11-21T22:05:11+09:00
draft: false
tags: ['next.js', 'tailwindcss']
summary: "next.js に tailwindcss v2.0 を導入したので備忘録として"
---

next.js に tailwindcss v2.0 を導入したので備忘録として導入方法を書き残します。前に v1.x を導入したときと手順が若干違った。

## 環境

yarn: 1.22.4

package.json

```json
{
  "dependencies": {
    "next": "10.0.2",
    "react": "17.0.1",
    "react-dom": "17.0.1"
  },
  "devDependencies": {
    "autoprefixer": "^10.0.2",
    "postcss": "^8.1.8",
    "tailwindcss": "^2.0.1"
  }
}
```

## 手順

### プロジェクトの作成

next.js のプロジェクトを作成します。

```bash
$ yarn create next-app
```

### 必要なパッケージのインストール

tailswindcss、postcss、autoprefixer をインストールします。

```bash
$ yarn add -D tailwindcss postcss autoprefixer
```

最新のフレームワークの多くは postcss を内部で使っているため、tailwindcss も postcss のプラグインとして使用することが推奨されています。なので今回はそれに従います。

### postcss のプラグインとして tailwindcss を追加

`postcss.config.js`に以下のように記述します。

```js
// postcss.config.js
module.exports = {
  plugins: ['tailwindcss', 'autoprefixer'],
};
```

また、以下のように tailwind のスタイルを css ファイルに記述します。デフォルトだと `styles/globals.css` が `pages/_app.js` で読み込まれているので、今回はこちらに記述します。

```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

この時点で tailwindcss が使えるようになっているはずです。`pages/index.js`を以下のように編集するとスクリーンショットのように tailwindcss を使用できていることが確認できます。

```jsx
// pages/index.js
export default function Home() {
  return (
    <div>
      <p className='text-center text-6xl text-blue-500 font-bold'>Hello Tailwindcss</p>
      <p className='text-center text-lg font-bold py-3'>in Next.js</p>
    </div>
  );
}
```

[![スクリーンショット](https://i.gyazo.com/5a3ca6870be5b72276d2a5473bb48c92.png)](https://gyazo.com/5a3ca6870be5b72276d2a5473bb48c92)

### purge を設定する

purgecss は使用していない css を削除することができます。tailwindcss のような css フレームワークを使う場合、大半のスタイルは使われることがないので、そういった無駄なものを削除して css ファイルのサイズを減らしてくれる偉い子です。tailwindcss には purgecss がすでに組み込まれているので、それを実際に適用します。

まずは設定ファイルを作成します。以下のコマンドを実行すると`tailwind.config.js`という設定ファイルが生成されます。

```bash
$ npx tailwindcss init
```

```js
// tailwind.config.js
module.exports = {
  purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};
```

`tailwind.config.js`の`purge`にビルドの対象になるファイルを指定します。

例

```js
// tailwind.config.js
module.exports = {
  purge: { content: ['./pages/**/*.{js,ts,jsx,tsx}'] }, // 修正
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};
```

これで purgecss の設定は終わりです。ちなみに tailwindcss の導入もこれでおしまいとなります。簡単でしたね。

## 最後に

はじめに書いたように以前導入したときは v1.x だったので、v2.0 で少し詰まりましたが公式ページが丁寧に書いてあったおかげで楽に導入できました。需要があれば Github にあげるかもしれませんが、参考記事にある next.js 本家の example とほとんど同じなので多分あげない。

質問は [@kmgk21444557](https://twitter.com/kmgk21444557) にお願いします。

## 参考記事

- [Installation - Tailwind CSS](https://tailwindcss.com/docs/installation)
  - tailwindcss 公式ページ。わかりやすい
- [next.js/examples/with-tailwindcss at canary · vercel/next.js](https://github.com/vercel/next.js/tree/canary/examples/with-tailwindcss)
  - next.js 公式の example。
- [Next.js に Tailwind CSS を導入する - パンダのプログラミングブログ](https://panda-program.com/posts/nextjs-tailwind-css)
  - v1.x の導入のときに参考にしたブログ。丁寧でわかりやすい。
