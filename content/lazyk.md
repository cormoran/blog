---
title: Lazy K　〜インタープリタの導入〜　 (Mac)
author: cormoran
date: 2015-02-26
categories:
  - Programing
tags:
  - language

---
先日はbrainfuckを触ってみましたが次は純粋関数型言語のLazy Kに挑戦してみます。今回はインタプリタと<del datetime="2015-02-26T15:11:20+00:00">インタラクティブシェルのインストール</del>（？）について書きます。<ins datetime="2015-02-26T15:11:20+00:00">次の記事に回します。</ins>

<!--more-->



LazyKについていろいろgoogleで調べたのですがやっぱり関数型だからなのかやさしい日本語情報がとても少ない気がしました。英語読めばいい話ですがやはり日本語の方が楽です.

## Lazy Kとは？

組み込み関数が I K S の３つしかない純粋関数型言語らしいです。Haskellなどの仲間で、C言語やbrainfuckなどの手続き型（命令型？）とは何か雰囲気が違う気がします。

概要は[Wiki][1]で。

詳細は[The Lazy K Programing Language][2]（英語）で。[ここ][3]で日本語訳がされています。

## 開発環境

・MacOSX Yosemite

・gcc 4.9

今回の話はLinuxでも同じようにできると思います。

## インタプリタ

自分で書いてみても面白そうですが始めなのでできたものを使わせてもらいます。以下の内容は主に、

[みゆっきのにっき　Lazy Kでhello World書いた][4]

を参考にしました。知り合いのサイトなのでなんか悔しいですがしますが自分にとっては一番わかりやすかった気がします。彼に遅れること１年での挑戦です。

　先ほどの The Lazy K Programing Language を読んでいくと The interpreter の項目でlazyというインタープリタがでてきます。最初こいつのソースがどこにあるかわからなかったのですが、みゆっきのにっきで参照されていた

[Math &#8211; 言語はどこまで小さくなれるか &#8211; (unlambda|iota|jot) のすすめ][5]

というサイトにリンクがありました。

[Index of /essie2/download/lazy-k][6]です。

ここのlazy.cppをダウンロードしてきてコンパイルします。上のサイトにある通り gcc（g++） でコンパイルしようとすると io.h がないとか言われてコンパイルできませんでした。パッチを当ててねって書いてあってそのデータも載っているのですが、windowsの実行ファイル形式のパッチしか当てたことがなかったので少し戸惑いました。(Unix系での)やり方はパッチのデータを適当な名前でlazy.cppと同じフォルダに保存して




patch < パッチのファイル名


とするとlazy.cppがパッチファイルに書かれたように修正されました。多分lazy.cppの名前を別の何かに変えていたり、内容をいじってしまっていたらうまくパッチを当てられないのでlazy.cppはオリジナルのままにしておかないといけないと思います。

　しかし、パッチを当ててコンパイルしてみても以下のようなエラーが出てきました。

~~~
lazy.cpp: In function 'int main(int, char**)':
lazy.cpp:562:58: error: taking address of temporary [-fpermissive]
     e = append_program(e, &File(stdin, "(standard input)"));
                                                          ^
lazy.cpp:575:48: error: taking address of temporary [-fpermissive]
     e = append_program(e, &StringStream(argv[i]));
                                                ^
lazy.cpp:586:42: error: taking address of temporary [-fpermissive]
    e = append_program(e, &File(f, argv[i]));

~~~

　なんか -fpermissive つけたくなる感じです。みゆっきのにっきにもさらりと書いてありましたが実際、



~~~
g++ -fpermissive -o lazy lazy.cpp
~~~



とするとさっきの error が warning にかわって（見た目ほとんど変わらないので気付きにくいですが）コンパイルできました。（-fpermissive オプションは適合しないコードを許可するとのこと）

これで

~~~
./lazy <lazy kファイル>
~~~

とするとそのファイルが実行できます。試しにlazy.cppがあったサイトの eg/ フォルダにある ab.lazy を落としてきて実行するとABAB&#8230;.と永遠に出力されます。(ctl+Cで終わる)

　インタープリタ取得完了。

## インタラクティブシェル

　と行きたいところですがなんかだらだら書いて長くなったので次回に回します。

[次の記事はこちら][7]

 [1]: http://ja.wikipedia.org/wiki/Lazy_K
 [2]: http://tromp.github.io/cl/lazy-k.html "Lazy k"
 [3]: http://legacy.e.tir.jp/wiliki?%cb%dd%cc%f5%3a%a5%d7%a5%ed%a5%b0%a5%e9%a5%df%a5%f3%a5%b0%b8%c0%b8%ecLazy_K
 [4]: http://solorab.net/blog/2014/04/09/hello-world-in-lazy-k/ "みゆっきのにっき"
 [5]: http://blog.livedoor.jp/dankogai/archives/51524324.html
 [6]: http://esoteric.sange.fi/essie2/download/lazy-k/
 [7]: http://blog.cormoran-web.com/2015/02/27/lazy-k-2-interactive/ "Lazy K に挑む　〜インタラクティブシェルの導入〜　 (Mac)"