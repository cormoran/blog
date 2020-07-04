---
title: "Mac ユーザーが Ubuntu に乗り換えたときの設定メモ"
date: 2018-10-29T05:41:34+09:00
---

最近ノートPCを Mac Book Pro Retina Late 2013 から ThinkPad T480s に変えた．

Linux を入れて環境構築していたら，いろいろあったのでメモ．
感想文や背景は[こちら](/blog/2018/10/thinkpad/)．

<!--more-->

## alt 左右で日本語/英語入力を切り替える

IM/IME の状態を覚えていられるほど賢くないので，トグルではなく，Macの英/かなと同じようにやりたい．

いろいろしらべたけど，IM を Fcitx に変えてしまうのが良い（デフォルトは iBus）．
Fcitx の設定画面からショートカットを設定できる．

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/144329185@N06/45605265941/in/dateposted-public/" title="Screenshot from 2018-10-29 05-54-23"><img src="https://farm2.staticflickr.com/1952/45605265941_a0b5371214_n.jpg" width="320" height="216" alt="Screenshot from 2018-10-29 05-54-23"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

iBus でやるなら，[USキーによる日本語入力をUbuntuでも快適に使う](http://yuki-tkd.hateblo.jp/entry/2016/04/21/001712)を参考に，Ubuntu のショートカット設定に `ibus engine 'xkb:us::eng'`, `ibus engine 'mozc-jp'` を追加する方法がある．
ただし，UbuntuのGUIの設定では，左右Alt単体のショートカットが設定できないっぽいので，GUIから適当に２つキーボード・ショートカットを作って，`dconf`を直接触って，`Alt_L`,  `Alt_R` を設定する必要があった．

これでいい感じに動くが，実は問題があって， mozc の初期状態が'直接入力'にハードコーディングされているせいで，ログイン後に毎回 mozc の設定から入力を日本語に変える必要がある．

## HiDPI な WQHD 液晶でいい感じに表示する

いままで Retina ディスプレイに毒されてきたので，FullHD の文字に耐えられないなぁと思って WQHD にした．が，Linux デスクトップ環境の HiDPI 対応はまだまだ微妙っぽい．

GNOME3 は現状整数倍スケーリングにしか対応していないらしいけど，ThinkPad T480s で 2倍スケーリングするとちょっと作業領域が小さくなりすぎる．
今までの Mac と同じ位のサイズ感になってほしくて，1.8倍くらいでスケーリングしてほしい．
[Arch Wiki](https://wiki.archlinux.jp/index.php/HiDPI)いわく，Wayland に変えたら experimental feature でできるとか，xrandr で実際より大きく出力して縮小するとか試したけど，experimental feature はフォントがギザギザで，xrandr は スクロールがちょっとカクッとなったり，外部ディスプレイにつなぐとバグる（これはちゃんとやれば直せる）．
Chrome は force-scale すれば とても良い感じになってくれるけど，ほかは GDK_SCALE 変えたりいろいろ試しても満足な方法がなくてつらさしかなかった（あまりわかってない）．

最終的には，[このサイトの記事](https://mensfeld.pl/2018/05/lenovo-thinkpad-x1-carbon-6th-gen-2018-ubuntu-18-04-tweaks/)と同じように，スケールは 100% にして，フォントのスケールを 1.8 にすることにした．
これだと，ウィンドウのボタンや，GNOMEのUIなどがとても小さくなるけど，ほかはとてもいい感じに動く．
どうでもいい部分が小さくなったのでこちらのほうがいいかもと思うくらい．
というわけでそうした．
Chrome だけは，/usr/bin/google-chrome-stable をいじって， `--force-device-scale-factor=1.8` を設定した．
ウィンドウサイズの関係で時々設定が潰れるけど，その場合は一時的に戻せばなんとかなる．

## できなかったこと

Mac では，３本指でウィンドウドラッグがとても便利だったのだけど，これは簡単にはできないっぽい．
やってる人もいるっぽいので暇があればやりたいけど，ThinkPadの物理ボタンがいい感じなので，そこまで必要じゃない気分になってきた．