---
title: MacでiBeacon発信
author: cormoran

date: 2014-10-14
categories:
  - Programing
---
MacをiBeaconの発信器にしてみました。

<!--more-->



　せっかくiPhone持ってるのだからiBeaconの発信器作って遊びたいなと以前から思っていたのですが、調べてみたところMacがiBeaconの発信器になるとの情報を発見しました。よくよく考えるとiBeaconはBluetooth4.0のBLE規格を使ってるので、それが使える計算機なら発信器にできるのも無理はありません。

　MacでiBeacon発信器には、調べた限りではBeaconOSXというMacアプリを作ってやるものとnode.jsを使ったものが手軽な方法として見つかりました。ちょっとNode.jsを触ってみたいという思いもあったので今回はNode.jsを使った方法を試してみました。

## Node.jsのインストール

　まずNode.jsとそのパッケージマネージャnpmをインストールします。自分はMacPortsを使いました。<a href="http://nodejs.org/" title="node.js" target="_blank">http://nodejs.org/</a>から.pkgファイルを落としてきてインストールする方法もあります。

~~~

sudo port install nodejs

sudo port install npm

~~~

## iBeacon

次にiBeaconを使うためのモジュール（と思われる）bleaconをnpmでインストールします。どこにインストールしたら良いのかNode.js初心者なのでよくわかってないのですがiBeacon専用にフォルダを作ってそこにインストールしたら良いようです。以下はホームディレクトリ下にiBeaconというなのフォルダを作って、その下にbleaconを入れる例。

~~~

mkdir ~/iBeacon

cd ~/iBeacon

npm install bleacon

~~~

以下のファイルをiBeacon下にbeacon.jsという名で作る

~~~

var Bleacon = require(&#8216;bleacon&#8217;);

var uuid = &#8216;AAAAAAAABBBBCCCCDDDDEEEEEEEEEEEE';

var major = 100;

var minor = 50;

var measuredPower = -59;

Bleacon.startAdvertising(uuid, major, minor, measuredPower);

~~~

## 実行

~~~

node ./beacon.js

~~~

これで発信します。（BluetoothをONにしておく）

受信側はiPhoneでBLUETUS for iBeaconというアプリを使いました。こいつは探索対象UUIDの初期値がAAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEEになっていたので上のプログラムを実行するとこのアプリで探知できるはずです。

下は受信結果です。

![](/img/2014/10/iBeacon_rec.jpg)

詳しいことはまた調べるつもりなのでこれを使って何か作るかもしれません。
