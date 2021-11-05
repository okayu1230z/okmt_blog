# Raspberry Piで完結する監視カメラシステム Compact-CameraAndStorage


## Compact-CAS (Compact-CameraAndStorage)

日本では、監視カメラシステムの設置は非常に高い

高齢化による空き巣の増加やコロナを考慮した外出自粛で監視カメラを設置する需要は増していると思う

さらに、昨今の人工知能分野の発展で画像さえあれば何か面白いことができてしまうのでできることならデータをたくさん蓄えておきたい

さて、日本の製品としての監視カメラシステムはどれくらい高いのか？

初期費用で数万飛ぶだけでなく、データを貯めようとするとクラウドを利用したりして月額利用料なども支払う必要がある

しかし、この大金の捻出はIoTの民主化で解決できる

今日は [Compact-CAS](https://github.com/okayu1230z/c-cas) を紹介したい

{{< admonition type=tip title="Compact-CAS (Compact-CameraAndStorage)" open=false >}}
This is Compact-CAS (Compact-CameraAndStorage), which provides cameras, storage, and even a web service to check the data. Based on this manual, you can inexpensively build a surveillance camera system for which you want historical data. All you need is as simple as a single Linux machine (e.g. Ubuntu, Raspberry Pi OS), a camera (e.g. sensor type, USB connected), and storage (e.g. SSD, HDD).
{{< /admonition >}}

こう書いてあるけど、簡単にいうと「Webカメラで動体検知して画像・動画をSSDに保存していく、ストレージ一体化の監視カメラ」のことである

機能を見ていこう

まずは Apache の list directory をそのまま用いているのだけど、日付ごとに directory が違うので、このままでも比較的ログは追いやすい

それでは、既存比較したものを表で見てみよう

○/△/x 

  Product  | マネジメントコンソール | Column 2 | Scalability | Cost |
---------------|----------|---------|---------|---------|
 Compact-CAS | Cell A-1 | Cell A-2 | ○ | ○
 これ | Cell B-1 | Cell B-2 | x | △
 これ | Cell B-1 | Cell B-2 | x | △
 これ | Cell B-1 | Cell B-2 | x | △
 これ | Cell B-1 | Cell B-2 | x | △

バイアスもかかってるだろうし、抜け漏れ勘弁

カメラ 1台、SSD 1台で Compact-CAS 運用する方法は上記のドキュメントの方にまとめてあるので、ここでは複数台同時に setting する方法を紹介する

## pssh を使った RPi の複数台セッティング

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
bv_ror_on_docker
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
[2] 23:18:56 [SUCCESS] ccas3
[3] 23:18:56 [SUCCESS] ccas2
[4] 23:18:56 [SUCCESS] ccas4
```

以上の結果、txtファイルの順番通りじゃないのは、コマンドの実行結果が返却された順番に出力されているからなのかなと思ってます。

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

## USBTemper

[USBTemper](https://www.amazon.co.jp/dp/B004FI1570)、一見ものすごく怪しい製品だけど、SwitchBotに比べて1,000yen安い


### Ubuntu 20.04 on RPi に hcitool をインストールする

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


## 終わりに

今回の自立型の監視カメラシステムの実装には難解なプログラミングも必要もない

このような基本的なIoT構築は個人レベルで簡単に行うことができる

ところで実家の畑に複数種類の動物が出現して野菜を荒らしているらしい...

これを踏まえてCompact-CASをベースに暗視できるようなセンサを用いて野外用監視カメラシステムを組んでみることを考えている



