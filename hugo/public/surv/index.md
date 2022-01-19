# システムにおける監視色々なソフトウェアをまとめてみた


## なぜ令和に新卒エンジニアがPerlを？

理由はシンプルで配属された部署のメインコンポーネントがPerlで書かれているからだな

Perl。全盛期は20年近く前。作者、Larry Wall（Perlプログラマの聖典、ラクダ本を書いたのもこの人。）

世の中はAWSやらGolangやらNext.jsやらOpenAPIやらクリーンアーキテクチャやらGraphQLやらで賑わっているけど、

こちとらオンプレでPerlで生JSで仕様書なくてスパゲッティおいしくてRESTだから！

楽しんでいこう

## 任意のPerlのバージョンを動かす

Macならperlbrewがある

```
$ curl -kL http://install.perlbrew.pl | bash
$ echo "source ~/perl5/perlbrew/etc/bashrc" >> ~/.zshrc
$ source ~/.zshrc
$ perlbrew --version
/Users/okmt/perl5/perlbrew/bin/perlbrew  - App::perlbrew/0.92


```


利用可能なPerlのバージョンを調べて、インストールしてバージョンを切り替えることで古いバージョンのPerlを使うことができる

```
$ perlbrew available
# perl
   perl-5.35.5   
   perl-5.34.0   
   perl-5.32.1   
   perl-5.30.3   
   perl-5.28.3   
   perl-5.26.3   
   perl-5.24.4   
   perl-5.22.4   
   perl-5.20.3   
   perl-5.18.4   
   perl-5.16.3   
   perl-5.14.4   
   perl-5.12.5   
   perl-5.10.1   
    perl-5.8.9   
    perl-5.6.2   
  perl5.005_03   
  perl5.004_05   


# cperl
  cperl-5.26.4   
  cperl-5.26.5   
  cperl-5.28.1   
  cperl-5.28.2   
  cperl-5.29.0   
  cperl-5.29.1   
  cperl-5.30.0-RC1   
  cperl-5.30.0

```

ちなみにperlbrewで公開されていないバージョンを使いたい場合、ソースコードからビルドするしかない

## Perlの文法まとめ




## テクニック的なものまとめ

## 監視ソフトウェアを並べてみる

fluentd 

logstash
k
kibana

grafana

## 最後に

後置ifとかねぇかなとか思う今日この頃

基本文法やらテクニック的なものは随時追加します

