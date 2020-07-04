+++
date = "2015-10-18T15:11:01+09:00"
title = "Raspberrypiでサーバーを作るときの設定など"
categories = ["Programing"]
tags = ["raspberrypi"]
+++


Raspberrypiを使って自宅サーバーを動かしているのですが、今回RaspberyPi2を買ったのでサーバーを移しました。また必要になるかもしれないのでメモ程度に書いておきます。

<!--more-->

## Raspianのインストール

* NOOBSを使うと楽
* apt-get update,upgradeしておく

## locale設定

* defaultはen-utf8として、ja-utf8を追加する（shellは英語がいいから）
* ja-utf8をデフォルトにするならshellを日本語対応のものにしないと文字化けする

## ユーザーの設定
* まず追加(adduser)
* piと同じグループに入れる
* piユーザー削除

## SSH設定
* /etc/ssh/sshd_config内
* ポートの変更
* 公開鍵方式にしたり、パスワードログイン禁止したり、IPでアクセス制限かけたり...
* パスワードログイン禁止は、公開鍵でちゃんと繋がることを確認してから設定する

## ファイアウォール
* ufwとか使う
* sshのポートをアクセス試行回数制限　など

## Apache2,PHP5,MySQL
  いろいろ設定（参考サイト参照）

## メール
  ssmtpを入れた
  /etc/ssmtp/…でアカウント設定など

ほかにもいろいろしたけど忘れた。

## 参考になるサイト

[屋根裏Linux](http://wings2fly.jp/yaneura/category/raspberry%20pi/): 全般

[Linearstar](http://log.star.glasscore.net) : ssmtp

[Make.](http://make.bcde.jp) : いろいろ,ufwとか