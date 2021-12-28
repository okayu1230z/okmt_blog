---
title: "Raspberry Piで完結する監視カメラシステム Compact-CameraAndStorage"
subtitle: "CCAS (Compact-CameraAndStorage)"
date: 2021-10-01T00:00:00+09:00
lastmod: 2021-10-01T00:00:00+09:00
publishDate: 2021-10-01T00:00:00+09:00
#author: "okmt"
#authorLink: "okmtLink"
#description: "okmtDesc"
license: "MIT"
draft: false

tags: [Rasberry Pi, c-cas, onpremise, Linux]
categories: [Diary]
featuredImage: "/ccas/images/ccas.jpg"
featuredImagePreview: "/ccas/images/ccas.jpg"

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

## Compact-CAS (Compact-CameraAndStorage)

高齢化による空き巣の増加やコロナを考慮した外出自粛で監視カメラを設置する需要は増していると思う

しかし、監視カメラの導入は初期費用で数万飛ぶだけでなく、データを貯めようとするとクラウドを利用したりして月額利用料なども支払う必要がある

さらに、昨今の人工知能分野の発展でデータさえあれば何か面白いことができてしまうのでできたりするのでデータはたくさん蓄えておきたい

この大金の捻出はIoTの民主化で解決できる

今日は安価に手元で監視カメラシステムを構築できるOSSの [Compact-CAS](https://github.com/okayu1230z/c-cas) を紹介したい

{{< admonition type=tip title="Compact-CAS (Compact-CameraAndStorage)" open=false >}}
This is Compact-CAS (Compact-CameraAndStorage), which provides cameras, storage, and even a web service to check the data. Based on this manual, you can inexpensively build a surveillance camera system for which you want historical data. All you need is as simple as a single Linux machine (e.g. Ubuntu, Raspberry Pi OS), a camera (e.g. sensor type, USB connected), and storage (e.g. SSD, HDD).
{{< /admonition >}}

こう書いてあるけど、簡単にいうと「Webカメラで動体検知して画像・動画をSSDに保存していく、ストレージ一体化の監視カメラ」のことである

値段はSSDを含めた全てのパーツを集めたとしてもおよそ12,000円程度だろう

簡単に全体構成を見ていこう

- カメラは[motion](https://motion-project.github.io/)を用いて動画の録画、動体感知、書き込みディレクトリなどを設定している
- ログ表示にはApacheのlist directoryをそのまま用いているのだけど、日付ごとに違うdirectoryに保存しているので、このままでも比較的過去のログは追いやすい
- Linuxのsystemdの設定で一定時間経過した映像の自動削除機能がある

既存のサービスと比較したものを表を作ってみた（雑に調べたので間違った情報も含みますが申し訳ありません。嘘を嘘と見抜けない人はインターネットを〜以下略）

### 既存サービスとの比較

比較観点は以下の5つを勝手に定義させていただいた

- Type: クラウド型SaaS/OSS/導入型（オンプレミスに製品やパッケージソフトウェアなどをメーカー主導で導入する方法）
- Management Console: 管理を助けるGUIが提供されているか（管理が容易にできるなら○、データ確認機能だけなら△、それもないならx）
- Scalability: システムを自由に拡張できるか（SaaSならAPIや自社ストレージへの自動データ保存機能などの拡張機能が充実しているか）
- Storage: データ保存（無制限ダウンロードや自社ストレージへの自動保存は○、制限付きダウンロード機能はx）
- Cost: 導入費用や月間使用料などの金銭的コスト

○: 優れている/△: どちらでもない/x: 優れていない 

  Product  | Type | Management Console | Scalability | Storage | Cost |
---------------|----------|---------|---------|---------|---------|
 [Compact-CAS](https://github.com/okayu1230z/c-cas) | OSS | △ | ○ | ○ | ○
 [safie](https://safie.link/) | クラウド型SaaS | ○ | ○ | △ | △
 [Eagle Eye](https://tocca.biz/product/product-814/) | クラウド型SaaS | ○ | △(※1) | △ | ?
 [ELMO QBiC CLOUD](https://www.ricoh.co.jp/service/elmo-qbiccloud/) | クラウド型SaaS | ◯ | △ | ○ | △
 [Panasonic BUSINESS](https://biz.panasonic.com/jp-ja/products-services/security_networkcamera/lineup) | 導入型 | △/x（※2） | △（※3） | ○ | x

※1: カメラがマルチベンダーシステムで選択可能なため△
※2: ローカルストレージに保存する設計/専用機器を導入
※3: 自由に設計できる可能性を残すがメーカー特有のソフトウェアパッケージを利用する部分は融通が効かないと考えられるため

Storageという観点はAI/Deep Learningなどで解析を行う際にデータ量が必要になるが、その際に全てのデータを保存できていることが望ましいのでデータ取得後分析したいなどの需要があれば重要である

一方で制限付きダウンロードを提供しているSaaSでもタイムラプス動画が70時間や100時間などのみで解析可能という手法もあるのでよく考えて導入されたい

safieは工事して導入/工事できないような場所に導入/来客をカウントしながらの監視/自宅への監視導入など様々なケースを想定したプロダクトを提供しており、小売・飲食などの小規模事例から工場のライン工に至るまでの大規模な導入ケースもある

ELMO QBiC CLOUDは[サービス説明書](https://www.elmo.co.jp/info/support/guide/security/ELMO_QBiC_CLOUD_manual.pdf)が公開されているがかなり丁寧で印象が良い

NTT Communicationsの[coomonita](https://www.ntt.com/business/solutions/enterprise-application-management/coomonita.html)とか便利そうだとサービスの中でSafie製品も取り扱ってるしどーゆー立場なのか分からないので表からは退けておいたやつが何個かある

雑な表だが表を作成したのは2021年7月ごろで調べた資料のバイアスもかかってるだろうし、抜け漏れなども勘弁されたい

### pssh を使った RPi の複数台セッティング

カメラ 1台、SSD 1台で Compact-CAS 運用する方法は上記のドキュメントの方にまとめてあるので、ここでは複数台同時に setting する方法を紹介する

pssh は SSH の並列実行できるコマンドで、Ubuntu や Mac、Windows で配布されている

これを使って4台のRPiに同時にCompact-CASをインストールしていこうと思ふ

PRi の setup は各自に任せる

自分は Ubuntu 20.04 LTS をインストールした microSD を RPi4 に差して、ネットワークを固定IPを振って ssh鍵認証、SSHポート変更、まで set up した状態からスタートする

Mac で brew が使えるなら以下で入る

```sh
$ brew install pssh
```

psshのファイルができたら、なんでも良いのでファイルを作ってそこにsshしたいホストを列挙していく

私は以下のファイルをccas.txtという名前で保存した

```
ccas1
ccas2
ccas3
ccas4
```

.ssh/configはこんな感じで用意した

```
$ cat .ssh/config
Host ccas1
  HostName 172.16.0.40
  User okmt
  Port 10022
  IdentityFile ~/.ssh/id_rsa_okmt
Host ccas2
  HostName 172.16.0.41
  User okmt
  Port 10022
  IdentityFile ~/.ssh/id_rsa_okmt
Host ccas3
  HostName 172.16.0.42
  User okmt
  Port 10022
  IdentityFile ~/.ssh/id_rsa_okmt
Host ccas4
  HostName 172.16.0.43
  User okmt
  Port 10022
  IdentityFile ~/.ssh/id_rsa_okmt
```

これで実行したいコマンドとccas.txtをpsshに渡すと、各RPiで並列的にコマンドが実行できる

```
$ pssh -h ccas -i "ls"        
[1] 23:18:56 [SUCCESS] ccas1
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
[2] 23:18:56 [SUCCESS] ccas3
...
[3] 23:18:56 [SUCCESS] ccas2
...
[4] 23:18:56 [SUCCESS] ccas4
...
```

以上の結果、txtファイルの順番通りじゃないのは、コマンドの実行結果が返却された順番に出力されているからなのかなと思う

余談だけど ccas1 は Ubuntu Desktop なので default で Desktop や Downloads が生成されているけど、ccas2-4 は Ubuntu Server なので初期状態の home directory には何もない

sudo は以下のコマンドのようにして、適当なファイルにパスワードを書いてそれを読み込ませる

```
$ cat passwd | pssh -h ccas -x '-tt' --inline-stdout -I "sudo whoami"
[1] 23:41:01 [SUCCESS] manager1
[sudo] password for okmt: 
root
[2] 23:41:03 [SUCCESS] ccas3
[sudo] password for okmt: 
root
[3] 23:41:03 [SUCCESS] ccas2
[sudo] password for okmt: 
root
```

まぁこれでなんとか無理やりsudoを使ったけど、.\*\_history に入るよりはマシな運用だと思って見逃されたい

用済みになったパスワード直書きファイルは必ず削除してください

てことでgithubの手順通りCompact-CASをインストールする

```
$ cat passwd | pssh -h ccas -x '-tt' --inline-stdout -I "sudo apt install -y v4l-utils"
```

### USBTemper

ここからは同じようにUSB接続できるセンサーから温度も測りたいという方に紹介したい

[USBTemper](https://www.amazon.co.jp/dp/B004FI1570)、一見ものすごく怪しい製品だけど、SwitchBotに比べて1,000yen安い


##### Ubuntu 20.04 on RPi に hcitool をインストールする

hcitoolをUbuntu 20.04に入れる際に若干詰まった

依存しているパッケージを導入してbluetoothに関するソフトウェアを適切に配置・設定する必要がある

```sh
$ sudo apt-get update
$ sudo apt-get install libdbus-1-dev libdbus-glib-1-dev libglib2.0-dev libical-dev libr    eadline-dev libudev-dev libusb-dev make
$ cd /usr/src/
$ sudo wget https://www.kernel.org/pub/linux/bluetooth/bluez-5.34.tar.xz
$ sudo xz -d bluez-5.34.tar.xz
$ sudo tar xvf bluez-5.34.tar
$ cd bluez-5.34
$ sudo ./configure --disable-systemd
$ sudo make
$ sudo make install
```

debian系だと同じように入れれると思う

精度は...ちょっと低いかも。常に+2度くらいの温度が計測される

[USB方向転換機](https://www.amazon.co.jp/gp/product/B01I1P14VE)とか使えば温度は逃がせそう

## IoTに関する情報収集

何かやろうと思ったことがあるときに踏み抜かれていないバグなどに遭遇し抜け出せずに失敗することもあるが、根気よく頑張れば成功したりもする

一般的な知識などに関してはとにかくggrか、[TECHFEED](https://techfeed.io/stories)などの情報サイトで情報でIoTのタグをフォローしておくとかは忙しい人からすると結構良いと思う

[SORACOM](https://soracom.jp/)などが主催するカンファレンスなどに参加する

資格とかだと[IoTシステム技術検定](https://www.mcpc-jp.org/iotkentei/)とか...？（高いし民間試験なので持っているメリットが高いとは思わないけれど）

## 終わりに

今回の自立型の監視カメラシステムの実装には難解なプログラミングも必要もない

このような基本的なIoT構築は個人レベルで簡単に行うことができる

ところで実家の畑に複数種類の動物が出現して野菜を荒らしているらしい...

これを踏まえてCompact-CASをベースに暗視できるようなセンサを用いて野外用監視カメラシステムを組んでみることを考えている

