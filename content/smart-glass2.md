---
title: Google Glass もどきその２
author: cormoran

date: 2014-11-17

categories:
  - Programing
  - 電子工作
tags:
  - ウェアラブル
---
　前回紹介した自作ウェアラブルグラスについてです。

<!--more-->



このメガネは大学の学園祭で展示に使用したのですが学園祭までに様々な予定が被り、当初予定していた機能はまだ実現できません。しかし、展示になりうるものをなんとか作った結果、一応完成形を示すことができるくらいにはなりました。

## 基本機能

・透過ウェアラブルディスプレイ

・Raspberry pi type B+使用、Debian系OS raspbian 搭載

・USBモバイルバッテリー電源

・拡張ポート：USBポート、GPIOポートなど(Raspberry pi type B+のもの)

## 回路関係

・TFT液晶M014C9163SPIとSPI通信

・ディスプレイバックライト on/off スイッチ

・（学園祭展示用に）押しボタンスイッチx5

## プログラム

<a href="https://github.com/cormoran/RokkoFestival" title="プログラム" target="_blank">プログラム置き場</a>

GPIOは[airspayce.comのbcm2835ライブラリ][1]を使用、フレームバッファの読み取り方やRaspberry piの設定関連は[こじ研][2]というサイトで詳しく説明がされているので参考にさせてもらいました。ライセンスは

・画面共有ライブラリ（プログラム置き場の/Library内）

　セマフォを利用。

・フレームバッファ表示アプリ

　フレームバッファを液晶に表示する。画面サイズ的にきついがGUIも表示可能

・テトリスアプリ

　テトリスができる

プログラムの使い方：

<ins datetime="2014-12-13T11:53:01+00:00">あらかじめ<a href="http://www.airspayce.com/mikem/bcm2835/index.html">airspayce.comのbcm2835ライブラリ</a>をリンク先のInstallationを参考にインストールしておく。</ins>プログラム置き場のURLをgitでcloneしてAppsのFB,Manager,RootApp,Tetris各ファイル内でmakeする。RootAppの実行ファイルを起動時にバックグラウンド起動するように設定。設定の仕方は[こじ研][3]参照。再起動する。液晶をつないでおけば液晶に端末がCUIが表示される。

## 画像

![4]
![5]
![6]
![7]

## 動画

学園祭展示に使ったテトリスです。

<iframe width="560" height="315" src="https://www.youtube.com/embed/MiLYOQ4eNDY" frameborder="0" allowfullscreen></iframe>

 [1]: http://www.airspayce.com/mikem/bcm2835/index.html
 [2]: http://www.myu.ac.jp/~xkozima/lab/raspTutorial1.html "こじ研"
 [3]: http://www.myu.ac.jp/~xkozima/lab/raspTutorial3.html
 [4]: /img/2014/11/IMG_0597.jpg
 [5]: /img/2014/11/IMG_0598.jpg
 [6]: /img/2014/11/IMG_0596-2.jpg
 [7]: /img/2014/11/IMG_0585-2.jpg
