---
title: "Hugoにテーマを適用し、自動ビルド後にNginxでデプロイするDocker環境を作る"
subtitle: "Apply the theme to Hugo & Create a Docker environment to automation build and deploy with Nginx"
date: 2021-08-28T00:00:00+09:00
lastmod: 2021-08-28T00:00:00+09:00
publishDate: 2021-08-28T00:00:00+09:00
#author: "okmt"
#authorLink: "okmtLink"
#description: "okmtDesc"
license: "MIT"
draft: false

tags: [Static Site Generator, Hugo, Docker, Onpremise]
categories: [Diary]
featuredImage: "/hugo/images/hugo.png"
featuredImagePreview: "/hugo/images/hugo.png"

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: true
code:
  copy: true
  # ...
math:
  enable: true
  # ...
mapbox:
  accessToken: ""
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # located in "assets/"
    # Or
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # located in "assets/"
    # Or
    # someJS = "https://cdn.example.com/some.js"
seo:
  images: []
  # ...
---

## 静的サイトジェネレータ（Static Site Generator）

SSGは入力ファイルから静的ページを構築するためのツール

表で紹介するものはMarkdownコンテンツを取り込み、テンプレートを適用してCMS用ですぐ利用できるもので、本 Blog では Hugo を使っている

| SSG Framework | Lang | Description |
| ------ | ---- |----------- |
| Hugo  | Golang | ナウいしビルド早い |
| Gatsby | JavaScript (React) | ナウいしテーマ多い |
| Jykill | Ruby | 古い |

表に知名度と自由度の高い Next.js Nuxt.js は入れてないヨ

動的 CMS には WordPress なんかがあって、これはユーザがサイトにアクセスするたびに必要に応じてページを構築する

静的 CMS は Adobe Dreamweaver や Movable Typed 等でこれは SSG と同じ仕組みで動いている

実は自分用のメモとかには Jykill 使ってて、ローカルに Git Repository 立ててメモを管理している

記事やメモを色んな Server から Origin にアップし続けるだけでドキュメントがウェブサービスで見れるようになるので便利だなー、と。

体感したことはないけど、Jykill は Hugo なんかよりビルド速度がかなり遅いらしい

Gatsby はフロントエンドエンジニアとの相性が良いように思える

### Hugoの利用

Hugo のインストールは[公式インストール方法](https://gohugo.io/getting-started/installing/)通り

よく使う hugo コマンドだけ軽くまとめた


| Command | Description |
| ------ | ----------- |
| hugo  | ビルドを行う |
| hugo check   | ビルドできるかチェックする |
| hugo server  | ビルドを行なってサーバを立てる |
| hugo server -p 8080 | ビルドを行なってポート指定してサーバを立てる |

### テーマの適用

公式サイトで紹介されている[hugo themes](https://themes.gohugo.io/)の中から好きなテーマを選び放題

テーマの適用方法は [公式Manual](https://hugoloveit.com/theme-documentation-basics/)通りでできそう

LoveIt という誰もが使いそうな Lovery な Theme があるので、このサイトでもそれを使っている

[Theme Loveit](https://themes.gohugo.io/themes/loveit/)


ただ、自分の場合は、`hugo serve`でエラーが出たので、(このサイト)[https://zhuanlan.zhihu.com/p/262906525]の内容を実装しないと hugo serve が実行できなかった。

Hugo は基本的には Markdown で記述できるが、Inline HTML も使えるので、拡張性にはさほど困らない。

[Markdown Syntax for LoveIt](https://hugoloveit.com/basic-markdown-syntax/)

数式表現がTEXでできるのは個人的にポイント高い

\\[ f(x)=\int_{-\infty}^{\infty} \hat{f}(\xi) e^{2 \pi i \xi x} d \xi \\]

$$ \ce{Hg^2+ ->[I-] HgI2 ->[I-] [Hg^{II}I4]^2-} $$

使いたい画像が png であることが多く、よく ImageMagick を使って jpg に変換する

    $ convert hogehoge.png -quality 100 hogehoge.jpg

## Docker でビルドしてデプロイ

{{< figure src="/hugo/images/docker.png" >}}

Hugo を Docker で扱うには、[こんなイメージ](https://hub.docker.com/r/klakegg/hugo/)もあるのでそれを利用して、Deployまで持っていけると思う

ただ、`hugo server`で実行した場合は、huge server なるもので配信しているのでこれを nginx で配信するようにしたい

hugo server は公式では「A high performance webserver」って紹介されていて、ほんまか？って感じやけど、同時に

> Many run it in production, but the standard behavior is for people to use it in development and use a more full featured server such as Nginx or Caddy.

ということでもあるのでこだわる人は Apache やら Nginx に乗り換えるのが良さそう

とりま Hugo が動く Dockerfile は発見

[Dockerfile | hugo-extended-docker](https://github.com/peaceiris/hugo-extended-docker/blob/main/Dockerfile)

これを元に作ってみる

### docker-compose によるビルド

出来上がった docker-compose はこんな感じ

[hugo_nginx](https://github.com/okayu1230z/okmt_blog)

```
version: '3'

services:
  hugo:
    build: ./hugo
    volumes:
      - ${PWD}/hugo:/src
  nginx:
    build: ./nginx
    depends_on:
      - hugo
    ports:
      - "8081:80"
    volumes:
      - ./hugo/public:/usr/share/nginx/html
```

hugo image で build し、nginx で出来上がった静的ファイルを配信している

これを使用方法は

```sh
$ docker-compose up
```

で hugo による build が走り、変更が反映されて Nginx で配信される

Build も Deploy も Theme には依存していないので任意の hugo project に使用できると思います。

hugo の Build 早いなぁ、Nginx の server 起動の方が時間かかってそう

FROMが複数あるDockerfileを docker docs で言われるところの[マルチステージビルド](https://docs.docker.com/develop/develop-images/multistage-build/)にはなっていないので注意すること

{{< admonition type=tip title="マルチステージビルド(multi stage build)" open=true >}}
マルチステージビルドはめっちゃ簡単にいうと Dockerfile に FROM が複数指定されていて、各レイヤをできる限り小さく保ち、効率的なビルド環境を構築すること
{{< /admonition >}}

こんな感じの記事を月一くらいで書けたらな、と思っているので良いなと思ったら記事のシェアとかTwitterをフォローしてください。

