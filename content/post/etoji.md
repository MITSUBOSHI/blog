---
title: "RubyのWebAssemblyを試してみた"
date: 2023-01-10T22:30:17+09:00
categories: ["ja"]
tags: ["ruby", "wasm", "tech"]
slug: "etoji"
draft: false
---

年始に育児の合間を縫ってRuby3.2で少しだけ遊んでみました。
上の子が絵本『[十二支のはじまり](https://www.amazon.co.jp/dp/4774604097)』を最近読んで、干支を認識し始めて来たので、それに関連したものを簡易的に作りました。

https://mitsuboshi.github.io/eto/

<!--more-->
~~※iOS/Android端末のブラウザでは上手く動作しないです~~
※2024-01に動くようになりました

## gem "etoji"

https://github.com/MITSUBOSHI/etoji


以下のように西暦から対象となる干支(十二支)のデータを取得するシンプルなgemです。
(`geto` はget + 干支の造語)

```ruby
require 'etoji'
Etoji.geto(year: 2023)

# => #<data Etoji::Jyunishi::Animal
#  number=4,
#  emoji="🐰",
#  character="卯",
#  character_hiragana_kun="う",
#  character_hiragana_on="ぼう",
#  animal_name_ja="兎",
#  animal_name_ja_hiragana="うさぎ",
#  animal_name_en="rabbit">
```

モダンなRubyを書いてみたく、`Data` クラス や パターンマッチを使ってみました。
パターンマッチはずっと使いたいと思っていたんですが、中々使う機会が無かったの、無理やり使いました(が、今回のケースはcase-whenで十分なことに後で気づいた)。

Steep(RBS)で型チェックを試してみました。
`Data` クラスのrbsファイルは公式にまだ存在しなかったり、case-inの記法を使うと『Syntax `case_match` is not supported in Steep』と `Ruby::UnsupportedSyntax` になったりとやや困りました。
typeprofでrbsファイルの雛形生成をして、untypedな型定義を修正していく体験は良かったです。

sorbet(RBI)の[Shopify/tapioca](https://github.com/Shopify/tapioca)のようにRails application向けに良い感じなrbsファイルを吐き出せたり、
ディレクトリ構成に沿って型ファイルを吐き出せるようになるとより嬉しいです。

### Refs
- https://github.com/tbpgr/eto
  - 既に似たgemが存在していたので、参考にさせてもらいました
- https://github.com/github/gemoji
  - 干支(十干・十二支)のデータの管理の仕方は、このgemの `db/emoji.json` 参考にしました
- [https://ja.wikipedia.org/wiki/干支](https://ja.wikipedia.org/wiki/%E5%B9%B2%E6%94%AF)

## WebAssembly

https://github.com/MITSUBOSHI/eto

```sh
$ curl -L https://github.com/ruby/ruby.wasm/releases/latest/download/ruby-head-wasm32-unknown-wasi-full-js.tar.gz | tar xfvz -
$ mv head-wasm32-unknown-wasi-full-js/usr/local/bin/ruby ruby.wasm
$ wasi-vfs pack ruby.wasm --mapdir /usr::./head-wasm32-unknown-wasi-full-js/usr --mapdir /etoji::./etoji -o docs/etoji.wasm
```

でetoji libraryを梱包してWASMを試すことが出来ました。
(本当はbundler経由でgem "etoji"を利用したかったが、手を抜きました。後で試そうと思います。)
最初公式repoのREADME等を読まずに何も考えず `ruby-head-wasm32-unknown-wasi-full` を利用していたため、ブラウザで必要な `rb-js-abi-host` が存在せず動かないなどお粗末なことをしました。
(https://github.com/ruby/ruby.wasm/tree/main/ext が必須)

「見た目は後で整えればいいだろう」「一旦、子どもに見せるか」と思って、iPhoneで動作しないことに気づき悲しくなりました。
macOS上のChrome/Safariでは問題なく動作はしていて、iOS * WebAssemblyで雑に調べるとメモリ関連で苦しんでいる記事をいくつか目にしたところで時間切れになったので、深追いしていません。

いつか仕事でもバックエンドとフロントエンドを共通ロジック(同一ソース)でバリデーションをさせるなどの用途でWebAssemblyを使うなどしてみたい気持ちです。

### Refs
- https://zenn.dev/koduki/articles/3619f53e8c0575
- https://github.com/kateinoigakukun/wasi-vfs/wiki/Getting-Started-with-CRuby
- https://github.com/ruby-syntax-tree/ruby-syntax-tree.github.io

## 追記(2024-01-06)

iOS/Androidで動作しなかった原因はwasmファイルのfetchを無駄に繰り返し行っていた点だった。
