---
title: 加速度センサーを使ってみた
author: cormoran

date: 2014-09-08
categories:
  - 電子工作
---

Strawberry Linuxの加速度センサーモジュール[MMA7455L（デジタル出力）][1]を使ってみました。このセンサーは昔買ってずっと放置していたものですが、最近は秋月などにより安価なものがたくさんあるようです。今回は出力値の補正などはせず、単に値を読み取ってLCDに表示するだけです。LCDについては[ここ][2]を参照。

<!--more-->

## MMA7455について

MMA7455LはSPI or I2C 出力の３軸加速度センサーです。3.3V動作なので使用時は注意が必要です。今回はI2C通信でPIC16F1827に接続してLCDに8bit精度で読み取り値を表示してみました。

## 動作テストの様子

<iframe width="640" height="360" src="https://www.youtube.com/embed/MypNXXZyoaE?feature=oembed" frameborder="0" allowfullscreen=""></iframe>

電源はLCDは電池から直接、マイコンと加速度センサは3.3V出力の３端子レギュレータを介しています。

詳細についてはI2C通信の関数群のモジュール化が完成したら書くかもしれません。

 [1]: https://strawberry-linux.com/catalog/items?code=12102
 [2]: http://blog.cormoran-web.com/?p=56 "ＬＣＤキャラクタディスプレイを使ってみた"