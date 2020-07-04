+++
date = "2016-12-18T20:17:26+09:00"
title = "Arduino互換の基板を自作してよくあるロボを動かす話(1)"

+++

[Kobe University Advent Calendar 2016](http://www.adventar.org/calendars/1881)の18日目です。

前回書こうとしてやめた話を書きます。実は時間がなくて完成してないので(1)としています。

<!--more-->

## 内容

- 基板を設計する
- 基板を発注する
- 基板に実装する
- プログラムを書く(次回？)

## 基板を設計する

　有名なツールは[EAGLE](https://cadsoft.io)、[KiCad](http://kicad.jp)など。他にもユニバーサル基板用の[PasS](http://www.geocities.jp/uaubn/pass/)、RSが作っている[DesignSpark PCB](https://www.rs-online.com/designspark/pcb-software-jp)などがある。

　EAGLEは商用ソフトで、無料版には基板サイズなどの制限がある。と言っても、趣味勢が作るような小さなものなら大体は問題なく作れる。よく使われるパーツは大体ライブラリが作られてその辺で公開されており、EAGLEはかつてはデファクトスタンダードだったと自分は認識している。

　KiCadはオープンソースで最近はやりな感じがある。EAGLEのライブラリをimportできるのでライブラリも大体はその辺からとってこられる。ちゃんと調べてないけど、今から始めるならとりあえずKiCad使えという感じがする。もちろんEAGLEのような制限はない。


## Arduino互換機として作る

　ArduinoIDEで（苦労なく）利用できるArduino互換基板として設計する場合、プログラムの書き込みの部分が重要。

  Arduinoの書き込みはシリアル通信のRX, TXのほか、DTRを使って行われる。DTRはマイコンのリセットピンに繋がっており、hackな感じの使い方がされている。
詳細は[ここのサイト](http://kutsushita-neko.cocolog-nifty.com/blog/2013/09/arduino-e10d.html)など参照。

　とりあえず、DTRに0.1uFかませてリセットに繋いでおけば良い。ネットで検索すると出てくるリセットから電源へのダイオードは、安定性は別としてなくても動く。

　マイコンの回路を作る場合、RX, TXはデバッグ用に出しておくだろうから、それに加えてリセットに0.1uf挟んだピンを出しておけば、USBシリアル変換回路を使ってArduinoIDEから使えるということ。

　その他の注意としては、マイコン、発振子の周波数はArduino既製品のどれかに合わせること。ブートローダーが自分でコンパイルできれば合ってなくても問題はないが、合わせておけばArduinoIDEから直接ブートローダを書き込める（書き込み器は必要）。また、Arduinoのライブラリを使う場合はライブラリ内で使うタイマーの関係でPWM出力に制限が出てきたりするので事前に調べておいて、使用するピン等を決めた方が良い。

　ArduinoIDEから使用する場合、ArduinoUnoなど自分のボードのマイコン・周波数にあう既存ボードを選択すれば良いが、自分のボードをArduinoIDEに登録することも可能。ArduinoIDE内のファイルをコピーしてちょこちょこ書き換えるとできる。どこかのサイトを参考にした気がするがさっき検索したところ良いサイトが見つからない。ArduinoIDEのホームページ探すと英語だけどありそうな気がする（別記事で書くべきか？）。さらに、ボード情報をネットに上げておいて、ボードマネージャから追加・更新というのも可能。ArduinoIDEは結構自由度が高い。[この辺り](https://www.arduino.cc/en/Hacking/HomePage)を見ると良さそう。

今年設計した基板たちを並べておく。ノイズ対策とか適当で晒すほどのものではない感じがあるが。

シリアル変換icを搭載していないタイプ

- [https://github.com/RokkoOroshi/CanMotorBoard2016](https://github.com/RokkoOroshi/CanMotorBoard2016)
- [https://github.com/RokkoOroshi/CanServoBoard2016](https://github.com/RokkoOroshi/CanServoBoard2016)

シリアル変換icを搭載したタイプ

- [https://bitbucket.org/cormoran707/tpip3sizemotorboard](https://bitbucket.org/cormoran707/tpip3sizemotorboard)

## 発注する

　あれこれして基板が設計できたとする。基板を作る方法は手軽なものならアイロンプリント法、感光基板で作る方法、加工機で作る方法、発注などがある自分は。昔はアイロンプリントしていたが、穴を開けるのが結構面倒だった。おすすめは発注。

　発注は海外業者に頼むのが一般的。どこがいいかは結構頻繁に変わるようだが、自分は[elecrow](http://www.elecrow.com)を使っている。日本語サイトなら[switch science pcb](https://www.switch-science.com/pcborder/)が安い（海外の[seeedstudio](https://www.seeedstudio.com)の代理をしているだけっぽいので安さ重視なら直接海外に頼めば良い）。

　価格はサイズによるが、50mm x 100mm までで10枚（同一基板）作ってくれて、\$10 + 送料\$5~くらい。重さによって送料が変わってくるが、為替等考えても2000円以内くらいで作れる。安い。ちなみに、頼んだ枚数より１、２枚余分に入っていることが多い。

　問題としては手元に届くのに２週間くらいかかることくらい。お金を積んで運送業者を変えればもっと早く届く。
　
## 作る

作る際に必要な部品は[秋月電子](http://akizukidenshi.com/catalog/default.aspx)、[RSオンライン](http://jp.rs-online.com/web/)等で買えば良いが、amazon, ebay, AliExpressなどで安く大量のパーツが買えたりもする（自分はその手の部品は買ったことがないのでまともなものが送られてくるかは知らない）。

作る際、はんだごては温度調整付きのものの方がbetter。半田付けは練習あるのみという感じ。

上で紹介したボードの完成写真

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/144329185@N06/31576869132/in/dateposted-public/" title="IMG_20161213_182538"><img src="https://c5.staticflickr.com/1/728/31576869132_53eb245e6d_n.jpg" width="320" height="240" alt="IMG_20161213_182538"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

設計ミスは力技でなんとかするのが一般的。

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/144329185@N06/31723873525/in/dateposted-public/" title="IMG_20161219_013927"><img src="https://c6.staticflickr.com/1/711/31723873525_eb6952246b_n.jpg" width="247" height="320" alt="IMG_20161219_013927"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
<a data-flickr-embed="true"  href="https://www.flickr.com/photos/144329185@N06/31350541660/in/dateposted-public/" title="IMG_20161219_013845"><img src="https://c5.staticflickr.com/1/562/31350541660_4fe2cd1eb0_n.jpg" width="256" height="320" alt="IMG_20161219_013845"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

かわいいアニメキャラ的なのをプリントするのが良いのだろうけど、そういうのに疎い（し、やったら真面目な方々から怒られそうな）ので某団体のロゴを突っ込んでおいた。こういうのを結構綺麗に入れられるのも発注の強み・楽しみ。

力尽きたので今日はこの辺りにしておく。

やろうとしていたのは、上で載せた大きい方のボードを使って、下のようなよくある車を動かす！という話。動かすのは一瞬で、動かすプログラムは書いたのだけど、制御画面なるものをelectronで作ろうとして、cssに凝りすぎて消耗したので放置している...。今年中に(2)を書きたい。

<a data-flickr-embed="true"  href="https://www.flickr.com/photos/144329185@N06/31686460886/in/dateposted-public/" title="IMG_20161201_185403"><img src="https://c7.staticflickr.com/1/378/31686460886_380efc8a0c_n.jpg" width="320" height="240" alt="IMG_20161201_185403"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

