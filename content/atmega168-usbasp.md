---
title: ATmega168pでUSBasp
author: cormoran
date: 2015-08-01
categories:
  - 電子工作

---
大学で所属しているロボット研究会で回路講座をすることになり、安価なAVRライタの量産が必要になったのでUSBaspを作ることにしました。今回はその試作時に起きた問題のメモ。


![UsbAsp](https://lh3.googleusercontent.com/-IW3yRNgI5QA/ViUUVeb5f0I/AAAAAAAAAGQ/CRlt_eK7fPc/s400-Ic42/usbasp.jpg)

<!--more-->

[USBasp][1]は公式にはATMega48とATMega88、ATMega8で作れるよと書いてありますが、今回はATMega168pを使いました。ATMega168pとATMega88は容量以外ほとんど同じ構造なのでターゲットの設定を変えるだけで作ることができました。ATMega88でいいところに容量が倍の168pを使っているので無駄な感じはしますが部室に大量にある168pを消費するには非常に好都合です。

## 環境

* Mac OSX Yosemite

* CrossPack for AVR 20131216

* usbasp.2011-05-28.tar.gz

## ATMega168p用にコンパイル

ATMega48とATMega88、ATMega8はバイナリで用意されていますが168p用はないのでソースからコンパイルします。

Readmeやmakefileが親切なので方法は読めばわかると思いますが、普通に

~~~
make main.hex
~~~

とすると以下のようなエラーが大量に出てきました。

~~~
usbdrv/usbdrv.h:485:5: error: variable 'usbDescriptorStringDevice' must be const in order to be put into read-only section by means of '__attribute__((progmem))'
 int usbDescriptorStringDevice[];
 ~~~

const にしなさいと言ってるのでusbdrv/usbdrv.h,usbdrv/usbdrv.c内のエラーになった部分に全てconstつけたらコンパイルが通りました。

あとは書き込むだけです。

fuse bitはATMega168pは88と同じなので low:0xff high:0xdd で大丈夫でした。

注意としては、USBaspは外部水晶発振器を使う設定になっているので一度fuseを書き込むとターゲットが外部水晶発振器モードになり、発振器をつけないとプログラムの書き込みができなくなります。発振器をつけておけばいいのですが、発振器なしで書き込むには先にプログラムを書いたあとfuseを書けばいいと思います。

## 動かす時のトラブル

USBaspは普通の5V IOポートに3.6Vツェナーダイオードをつけて簡易レベル変換することでUSBの仕様(D+-が3.3V)に合わせています。今回は手元に3.3Vツェナーダイオードしかなかったのでそれを使ったのですがPCが認識してくれませんでした。色々試した末、ツェナーダイオードを外すことで安定して認識し、書き込みを行うことができました。[千秋ゼミ][2]によるとツェナーダイオードは低電圧域での定電圧特性があまり良くないらしく、3.3VのものではAVR側で電圧レベルが足りない？か何かが起こってうまく通信できなかったようです。3.6Vにすれば大丈夫かもしれません。今は5Vのままで使っていますが3.3V仕様のところに5Vで入れるのは少し心配（気分が悪い）です...;

 [1]: http://www.fischl.de/usbasp/
 [2]: http://www-ice.yamagata-cit.ac.jp/ken/senshu/sitedev/index.php?AVR%2Fnews25a
