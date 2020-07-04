+++
date = "2014-08-31T03:35:07+09:00"
draft = true
title = "OpenCV"
categories = ["Programing"]

+++

MacでOpenCVを使ってみました。

<!--more-->

インストールはMacPortsを使って

~~~
$ sudo port install opencv
~~~

以外と時間がかかります。

とりあえず[http://opencv.jp/](http://opencv.jp/)からサンプルコードを取ってきて試してみました。

メニューバーのSAMPLE CODEにあります。
今回試してみたのはコーナー検出とエッジ検出。
サンプルコードはc++のものをそのままコピーして適当な.cppファイルに保存し、

~~~
$ g++ `pkg-config --cflags --libs opencv` hogehoge.cpp
~~~

でコンパイルできます。（多分）
以下は適当に探して見つけたサッカーボールの画像でのテスト。画像元は[こちら](http://www.freepik.com/free-vector/plain-soccer-ball_518796.htm)。

エッジ検出

![エッジ検出](/img/use-opencv/edge.png )


コーナー検出

![コーナー検出](/img/use-opencv/corner.png)

画像が良いこともありますがうまく検出してくれていますね。二足歩行ロボットで使うのが目標です。
