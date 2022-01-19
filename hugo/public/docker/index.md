# Docker の基礎と学び方


Dockerをある程度学んだので利用するメリット、最低限知っておきたい使い方と参考書物の感想をまとめてみたのでそれらを紹介する

個人的にDockerが結構好きというのもあるが、初めて使ったときその手軽さにかなり驚いたのを覚えていて、本記事が布教に役立てばと思っている。

<br>

## Dockerとは？

Docker とはコンテナ化仮想化アプリケーション実行環境プラットフォームである。

[公式サイト](https://www.docker.com/)から抜粋すると、Dockerは下のように説明されている。

{{< admonition type=tip title="Docker" open=false >}}
Docker takes away repetitive, mundane configuration tasks and is used throughout the development lifecycle for fast, easy and portable application development - desktop and cloud. Docker’s comprehensive end to end platform includes UIs, CLIs, APIs and security that are engineered to work together across the entire application delivery lifecycle.
{{< /admonition >}}

- 高速で簡単でポータブル（fast, easy and portable）なアプリケーション開発環境として、開発ライフサイクル(development lifecycle)全体で利用できるよ
- 反復的な構築作業(mundane configuration tasks)から解放させてくれるよ
- DockerというプラットフォームはUI, CLI, API, セキュリティなどあらゆるデリバリーライフサイクルを包括的に提供できるよ

欲張りすぎな説明に見えるかもしれないが、言い過ぎではない。

### よく取り上げられるメリット

- **Infrastructure as Code**: インフラをコード化することで、その冪等性を利用してコピーやスケールが用意になるなどのメリットがある
- **軽量**: DockerはVMに比べるとサイズ的に軽量で、実行時に必要なシステムリソースも少ない
- **コスト**: IaCにより環境構築などのイニシャルコスト、冪等性によりランニングコストを抑えることができる。
- **ポータブル**: アプリケーション実行環境を配布しやすい。配布に特化したDocker Hubというネットワークシステムもある。
- **DevOps**: スクラップ&ビルドも用意で、日々のリリースサイクルがスムーズになる

以上の5つは完全に独立しておらず、関係しながらも互いに成立している

##### ポイント1: IaC(Infrastructure as Code)

コスト・スピード・リスクの観点からメリットが語られることが多い

- コスト: 繰り返される手作業を排除して、エンジニアが他の作業に集中できる
- スピード: インフラを自動がすることで高速な実行が期待できる
- リスク: 人的ミスを排除することで信頼性が向上する

IaCを活用しインフラを構成するツールの総称をCCA(Continuous Configuration Automation)と呼び、代表的なものにはAnsibleやChefなどがある

##### ポイント2: 軽量

VMに比べて実行に必要な必要なシステムリソースが少なく、実行環境を選ばない

[Alpine](https://hub.docker.com/_/alpine)や[Debian Slim](https://hub.docker.com/_/debian)などの軽量イメージを元にImageを作成したりマルチステージビルドと組み合わせることでDockerの中でもさらに軽量で安全な実行環境を用意することができる

##### ポイント3: コスト

IoCの時に紹介したようにDevOps的な観点からもコストメリットがある

また、開発環境をそれぞれで構築する必要がなく、Docker Engineが動いているOSであればすぐに環境を構築できるという点でも人的リソースを削除できると考えられる

##### ポイント4: portable

開発中のソフトウェアを他のエンジニアと共有したり、

ソフトウェアを面倒な環境構築をさけてサクッと試してもらいたい時などはもはや必須の技術といえるんじゃなかろうか

無料だし（なんか最近Docker Desktopの制約付き有料化の流れもあるけど）

自分は仕事仲間とアプリケーションを共有したり、試したいOSSがあったりすると必ずと言っていいほどDockerを利用している

そのportableな性質を利用してマイクロサービスを開発するときにも利用される

##### ポイント5: DevOps

DevOpsはソフトウェア開発と運用が密接に連携することにより、柔軟かつスピーディーにシステムを提供するための考え方でCI/CDというプラクティスが有名

- CI(Continuous Integration): ソースコードを変更するたびにビルドと単体テストを実行することで不整合を早期に発見し、開発者の定常作業の効率化を見込むことができる
- CD(Continuous Delivery): CIを完了したアプリケーションを定期的にテスト環境へ配置し、常に動作可能な最新状態のアプリケーションを維持するプロセスを自動化すること。

本番環境にリリースするプロセスも自動化するCD(Continuous Deploy)という考え方もある

DockerはCI/CDとの相性がよく、Jenkinsなどのビルドパイプラインツールと組み合わせて使われる

##### 個人的な推しポイント: Orchestration toolの存在

オーケストレーションツールのKubernetes (a.k.a k8s)など、コンテナ化仮想化アプリケーションの管理を行うツールの存在がある

k8sを簡単に説明すると複数のDocker containerを管理する仕組みを提供しているソフトウェアである。

Dockerで定義したアプリケーションを簡単にスケール・管理することができる。

管理しているContainerがシャットダウンしても自動で定義された状態を保とうとする自己回復機能などがある。

<br>

## これだけ覚えておきたいDockerの使い方

- **Docker Image**: Docker Containerのベースとなるアーカイブパッケージのこと。[Docker Hub](https://hub.docker.com/)などで配布されている
- **Docker Container**: Docker Imageを元に作成するアプリケーション実行環境。Dockerfileを用いてDocker Imageを拡張できる

### Dockerfile

配布されているDocker ImageをベースにDockerfileに定義した内容で新しいDocker Imageのビルドを行える

Dockerfileに記述できる命令の一部を紹介

| Command | Description |
| ------ | ----------- |
| FROM | Dockerfile のベースイメージを設定 |
| LABEL | ラベルの設定。MAINTAINER とかも時が経ちこれで指定させるようになりました |
| ADD | イメージへファイルをコピー。他の選択肢として COPY や RUN curl なども選択肢にある（詳しい違いは省略） | 
| COPY | イメージへファイルをコピー |
| RUN | 指定された命令とコンテナ内で実行 |
| CMD | コンテナ起動後に指定された命令を実行 |
| ENTRYPOINT | コンテナ起動時に実行される実行可能ファイルを設定 | 
| ENV | イメージ内の環境変数を設定し、記述以降参照可能 |
| EXPOSE | Dockerに対し、プロセスが指定されたポート、もしくはポート群で待ち受けを行うことを指定 |
| WORKDIR | これ以降のRUN、CMD、ENTRYPOINT、ADD、COPYで使われる作業ディレクトリを設定 |

Dockerfileは以下のように定義する

```text
FROM debian

RUN apt update && apt install -y cowsay fortune
COPY entrypoint.sh /

ENTORYPOINT ["/entrypoint.sh"]
```

entrypoint.shという名前のbashを以下のように定義する

```bash
#!/bin/bash
if [ $# -eq 0];then
  /usr/games/fortune | /usr/games/cowsay
else
  /usr/games/cowsay "$@"
fi
```

ではターミナルで上記ファイルを配置したディレクトリまで移動して以下を実行してビルドする

`$ docker build -t cowsay .`

作業ディレクトリ配下のDockerfileののビルドが行われ、ローカルにcowsayという名前のDocker Imageが保存される

以下のコマンドでローカルに保存されたイメージは確認できる

```bash
$ docker images
REPOSITORY                         TAG       IMAGE ID       CREATED              SIZE
cowsay                             latest    a1ab28168777   About a minute ago   191MB
```

そしてDocker Containerの立ち上げpenDialogは以下のように実行する

`$ docker run cowsay`

```bash
$ docker run cowsay
 ________________
< Chess tonight. >
 ----------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


引数を変えながら動きを見てみる

```bash
$ docker run cowsay 楽しいDocker
 __
< 楽しいDocker  >
 --
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

ここでは牛のお話内容が変わるだけの分岐をbashで実行しているが、引数をtestとするとテストを実行したりするようにするような利用ケースが考えられる

##### Multi-Stage build

Dockerの公式ドキュメントにGolangの環境構築例が取り上げられていたので、そのまま持ってくる

以下のようなDockerfileを用意する

```text
FROM golang:1.16 AS builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html
COPY app.go    ./
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app ./
CMD ["./app"]
```

ビルド・実行を同じDockerfileで行うように設定することをマルチステージビルドという

「FROM golang」のイメージでビルドして、「FROM alpine」ではappを実行している

特徴的なのは10行目のCOPYでbuilderという名前をつけたImageでの成果物を2つ目のImageにCOPYするのを行なっている

### よく使うDockerコマンドまとめ

debianコンテナを起動し、bashでログイン（-it でインタラクティブな端末で接続）

`$ docker container run -it debian /bin/bash`

current directoryをweb server の公開directoryにmountして起動する（--name でコンテナ名前を指定して起動、-v でローカルディレクトリのマウント、-p でポートの指定、ちなみに、-d でバックグラウンドで実行）

`$ docker container run --name okmt-httpd -p 8080:80 -v "$PWD:/usr/local/apache2/htdocs/ httpd:2.4"`

コンテナ一覧を表示（-a optionでexitedなcontainerも表示）

`$ docker container ls`

コンテナ削除

`$ docker container rm <Container ID>`

停止したコンテナIDを全て表示

`$ docker container ls -aq -f status=exited`

つまり以下のようにすると停止したコンテナを全て削除できる

`$ docker continaer rm -v $(docker container ls -aq -f status=exited)`

コンテナの詳しい情報（引数、状況、ネットワーク構成）を表示

`$ docker container inspect okmt-httpd`

コンテナ内で変更されたファイルのリストを表示

`$ docker container diff okmt-httpd`

コンテナのログを表示（-f で表示し続ける)

`$ docker container logs okmt-httpd`
                                             
### Docker Composeについて

Docker Composeは複数のコンテナを定義し実行するツールで、yamlファイルを入力とする。

例えばNginxでホストにあるhtmlディレクトリにあるhtmlを配信するような設定のdocker-compose.ymlの記述は以下のように書ける

```yaml
version: '3'

services:
  nginx:
    container: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

このdocker-compose.ymlが存在するディレクトリにnginxで配信したい.htmlファイルを用意したhtmlというディレクトリを用意し、以下のコマンドを実行する

```
$ docker-compose up
```


記述方法は是非[公式](https://docs.docker.jp/compose/toc.html)などを確認してほしい

ローカルのファイルをマウントしたりアプリケーションをカスタマイズ実行しようとするとdockerコマンドの引数が長くなってくると思うのでComposeを活用しよう

<br>

## 参考書の紹介 

書籍に書いてある内容とこの記事の中身はそこまで関係ないので以下の感想を読んでみて気になる人は実際に読んでみてね

[さわって学ぶクラウドインフラ docker基礎からのコンテナ構築 | 日経BP](https://www.amazon.co.jp/gp/product/B089VZXX63)

図表や説明などは分かりやすく、初学者にとってはとっつきやすい本だと感じた

タイトルは"クラウド"に限定されているように見えるが中身はタイトルほど"クラウド"に限定していないように見える（そもそも Docker が"クラウド"のためだけのインフラでないことは明確ですが）

Dockerに関する作業は全てAmazon EC2のUbuntu上で行うが、ローカルで行っても差分がない内容ではある

本の後半ではk8sに関する内容がまとめられていて、とてもわかりやすい説明だったと思う

[Docker | O'REILLY](https://www.amazon.co.jp/gp/product)

難易度としては上の本よりも数段上にあり、Docker や Docker の構成要素を少し深くまで学習することができた

Ansible や Trition などの Docker が関連するソフトウェアも幅広く紹介していて、知識の幅も広がる

一方でポイントを絞って学ぶという観点では範囲が広すぎるので、
「俺はDockerを使うのに必要最小限の知識だけあればいいぜ！」という人には向いていない本かもしれない

幅広く、でも周辺ソフトウェアも取りこぼしたくなかったので、上の本と組み合わせて学習することに

## 総括

ちなみにGoogleは全ての開発をDockerでやっている

Containerを利用できるクラウドサービスも増えてきているしやk8sもアツいし、

今後、Dockerはアプリケーションエンジニアの必須な技術スタックになってくると思う


