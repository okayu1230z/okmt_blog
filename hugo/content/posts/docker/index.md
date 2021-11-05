---
title: "Docker を学んでみよう（本読み感想含む）"
subtitle: "Let's learn Docker (reading books impression)"
date: 2021-09-01T00:00:00+09:00
lastmod: 2021-09-01T00:00:00+09:00
publishDate: 2021-08-29T00:00:00+09:00
#author: "okmt"
#authorLink: "okmtLink"
#description: "okmtDesc"
license: "MIT"
draft: false

tags: [Certification, LPIC, Linux]
categories: [Diary]
featuredImage: "/docker/images/docker.png"
featuredImagePreview: "/docker/images/docker.png"

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

## Docker と勉強方法

Docker とはなんぞや？

{{< admonition type=tip title="Docker" open=false >}}
公式のドキュメント
{{< /admonition >}}

開発中のソフトウェアを他のエンジニアと共有したり、

プロトタイプをサクッと試してもらいたい時などはもはや必須といえるんじゃなかろうか。

無料だし（なんか最近有料化の流れもあるけど）

この記事では、Docker の使い方と（ちょっと）仕組みと（ちょっと）周辺技術をまとめてみたのでそれらを紹介する

書籍に書いてある内容とこの記事の中身は違うので気になる人は実際に読んでみてね

### 参考書の紹介 

[docker docs](https://docs.docker.com/)

公式の文章は重要なのですが、今回は言葉の定義を確認したいときなどを中心に辞書的に使いました。

[さわって学ぶクラウドインフラ docker基礎からのコンテナ構築 | 日経BP](https://www.amazon.co.jp/gp/product/B089VZXX63)

タイトルは"クラウド"に限定されているように見えるが中身はタイトルほど"クラウド"に限定していないように見える

そもそも Docker は"クラウド"のためだけのインフラでないことは明確ですが、
それだけにこのタイトルの場合はもう少しクラウド取り扱っていても良かったのかなぁと

内容的に少しだけ引っかかるところがあったのでそれは本記事の最後にまとめておくわね。

図表は分かりやすかった！

[Docker | O'REILLY](https://www.amazon.co.jp/gp/product)

難易度としては上の本よりも数段上にあり、Docker や Docker の構成要素を少し深くまで学習することができた

Ansible や Trition などの Docker が関連するソフトウェアも幅広く紹介していて、知識の幅も広がる

一方でポイントを絞って学ぶという観点では範囲が広すぎるので、
「俺は使うのに必要最小限の知識だけあればいいぜ！」という人には向いていないかも知れない本

幅広く、でも重要なところは取りこぼしたくなので、上の本と組み合わせて学習することに

結果、1ヶ月もせずあんなに知らんかった Docker の理解が確実に進んだのでこの学習方法は合う人には合うかも

## Docker まとめ

Docker を使うと、host OS との resource 共有、Container の Portability、軽量なので実動環境そのままの分散システムを emulate できたりと、メリットたくさんあるよ

Docker は scale-out が容易に行えるので micro service も use case として挙げられるよ

### Dockerfile からイメージを作成

Dockerfileに記述できる命令は以下のようになっている

| Command | Description |
| ------ | ----------- |
| FROM | DOckerfile のベースイメージを設定 |
| LABEL | ラベルの設定。MAINTAINER とかも時が経ちこれで指定させるようになりました |
| ADD | イメージへファイルをコピー。他の選択肢として COPY や RUN curl なども選択肢にある（詳しい違いは省略） | 
| COPY | イメージへファイルをコピー |
| RUN | 指定された命令とコンテナ内で実行 |
| CMD | コンテナ起動後に指定された命令を実行 |
| ENTRYPOINT | コンテナ起動時に実行される実行可能ファイルを設定 | 
| ENV | イメージ内の環境変数を設定し、記述以降参照可能 |
| EXPOSE | Dockerに対し、プロセスが指定されたポート、もしくはポート群で待ち受けを行うことを指定 |
| WORKDIR | これ以降のRUN、CMD、ENTRYPOINT、ADD、COPYで使われる作業ディレクトリを設定 |

引数の動きを確認する

```Dockerfile
FROM debian

RUN apt update && apt install -y cowsay fortune
COPY entrypoint.sh /

ENTORYPOINT ["/entrypoint.sh"]
```

```bash
#!/bin/bash
if [ $# -eq 0];then
  /usr/games/fortune | /usr/games/cowsay
else
  /usr/games/cowsay "$@"
fi
```

#### docker build

`docker build -t test/cowsay-docker file`

### docker image command

`$ docker image ls` : 保存されているイメージを全て表示

`$ docker image history okmt_nginx:lastet` : okmt_nginxイメージのレイヤを表示する

`docker image rmi -f $(docker image ls -a -f "dangling=true" -q)` : <none>:<none>となっているどうしようもないイメージを削除する

`$ docker image prune` : 上と同じ効果

### docker container command

`$ docker container run -it debian /bin/bash` : debianコンテナを起動し、bashでログイン（-it でインタラクティブな端末で接続）

`$ docker container run --name okmt-httpd -p 8080:80 -v "$PWD:/usr/local/apache2/htdocs/ httpd:2.4"` : current directory を web server の公開 directory に mount して起動する（--name でコンテナ名前を指定して起動、-v でローカルディレクトリのマウント、-p でポートの指定、ちなみに、-d でバックグラウンドで実行）

`$ docker container ls` : コンテナ一覧を表示

`$ docker container ls -aq -f status=exited` : 停止したコンテナIDを全て表示

`$ docker continaer rm -v $(docker container ls -aq -f status=exited)` : 停止したコンテナを全て削除（上記コマンドを利用したいなら alias 貼っとくのが良いかも？自分は docker-rm-exited で登録してる）

`$ docker container inspect okmt-httpd` : コンテナの詳しい情報（引数、状況、ネットワーク構成）を表示

`$ docker container logs okmt-httpd` : コンテナ内で起きたことのリストを表示

`$ docker container diff okmt-httpd` : コンテナ内で変更されたファイルのリストを表示

`$ docker container logs okmt-httpd` : コンテナのログを表示（-f で表示し続ける）

## 総括

勉強よりもまとめることの方がしんどかった！

で、これをちゃんと学んでみて確信に変わったのが Docker はやはりインフラエンジニアのやることでなく、

アプリケーションエンジニアとしての素養として身につけておいて良いってこと。

### さわって学ぶクラウドインフラ（Kindle版）の気になったところ

いずれも細かいことであります！

{{< admonition type=tip title="気になる点 1" open=false >}}
docker container ls と docker ps がある -> そうだね

この本では短いコマンドを使いたいので docker ps を使う -> まぁ分かる

docker image ls を使う -> その流れだと使うのは docker images では？
{{< /admonition >}}


{{< admonition type=tip title="気になる点 2" open=false >}}
これは誤植かと思いますが、図表4-6の「-v "SPWD":/usr/local/apache2/htdocs」は
「-v "$PWD":/usr/local/apache2/htdocs」かも知れません。
{{< /admonition >}}

