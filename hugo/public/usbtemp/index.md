# Raspberry Pi に USBTemper を刺して温度を測る


## C-CAS (Compact-CameraAndStorage)

日本では、監視カメラシステムの設置は非常に高い

高齢化による空き巣の増加やコロナを考慮した外出自粛で監視カメラを設置する需要は増していると思う

さらに、昨今の人工知能分野の発展で画像さえあれば何か面白いことができてしまうのでできることならデータをたくさん蓄えておきたい

さて、日本の製品としての監視カメラシステムはどれくらい高いのか？

初期費用で数万飛ぶだけでなく、データを貯めようとするとクラウドを利用したりして月額利用料なども支払う必要がある

しかし、この大金の捻出は自ら既存のIoT部品を組み合わせることで解決できる

過去に紹介した [C-CAS](https://github.com/okayu1230z/c-cas) とも相性が良さそう


## 検証

結局、検証をちゃんと行わないと

## Ubuntu 20.04 on RPi に hcitool をインストールする

```sh
$ sudo apt-get update
$ sudo apt-get install libdbus-1-dev libdbus-glib-1-dev libglib2.0-dev libical-dev libreadline-dev libudev-dev libusb-dev make
$ cd /usr/src/
$ sudo wget https://www.kernel.org/pub/linux/bluetooth/bluez-5.34.tar.xz
$ sudo xz -d bluez-5.34.tar.xz
$ sudo tar xvf bluez-5.34.tar
$ cd bluez-5.34
$ sudo ./configure --disable-systemd
$ sudo make
$ sudo make install
```






