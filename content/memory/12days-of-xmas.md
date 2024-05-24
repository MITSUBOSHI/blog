---
title: "クリスマスの12日間"
categories: ["ja"]
tags: ["js", "tech"]
slug: "12days-of-xmas"
date: 2024-05-25T01:00:00+09:00
draft: false
---

GW期間中に暇を見つけてGitHub Pagesに新たにtoy siteを公開しました。
昨年のクリスマスシーズンに子どもたちと一緒に『[クリスマスの12日間](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AA%E3%82%B9%E3%83%9E%E3%82%B9%E3%81%AE12%E6%97%A5%E9%96%93)』の歌や絵本を楽しみました。
ただ私は歌詞を覚えるのがすこぶる苦手であり、次のシーズンまで覚えてられる気がしないので簡単なクイズ要素を含むものにしました。
　　
https://mitsuboshi.github.io/12days-of-xmas/
　　
技術的に面白みのある工夫は特にありません。
ビルドツールにViteを使ったのですが、唯一後述の点がハマりどころでした。

▼公式docsからの引用 [^1]
```
> GitHub Pages
> 1. vite.config.js で base を正しく設定してください。
```
　
カスタムドメインとそれ以外で `base` に設定すべき値も異なります。
開発時のローカル環境とGitHub Pagesで設定値を分ける必要があったため、最終的にはbuild時に環境変数 "GITHUB_PAGES" を渡して動的設定をするようにしました。[^2]

```js:vite.config.ts
  base: process.env.GITHUB_PAGES
    ? "/12days-of-xmas/"
    : "/",
```

公式docsにGitHub Pagesに関する記載があるのは大変助かりました。


[^1]: https://ja.vitejs.dev/guide/static-deploy#github-pages
[^2]: https://github.com/MITSUBOSHI/12days-of-xmas/blob/e328966b4b0f8b641e658900cd04b5fc5f88e270/vite.config.ts#L7-L9