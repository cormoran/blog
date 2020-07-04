---
title: Brainfuck インタープリター in c++
author: cormoran
date: 2015-01-09

categories:
  - Programing
tags :
  - language
---

周りで Brainfuck が流行っているよう（？）だったので自分も参入してインタープリターを書いてみました。

<!--more-->

## Brainfuckとは？

命令が８個しかないプログラミング言語です。詳細は[Wikipedia][1]参照。簡単に説明すると・・・

0で初期化された少なくとも30000個のバイト配列とポインタがあり、ポインタは配列の先頭を指している。この配列に対し以下の８つの命令のみが使える。

  * > : ポインタのインクリメント
  * < : ポインタのディクリメント
  * + : （ポインタがさす）メモリの値を1増やす
  * &#8211; : メモリの値を1減らす
  * . : メモリの値を出力に書き出す
  * , : メモリに入力から1byte読み込む
  * [ : メモリの値が0なら対応する ] （の直後）に飛ぶ
  * ] : メモリの値が0でないなら対応する [ （の直後）に戻る/li> </ul>

## Interpreter

[interpreter](https://github.com/cormoran/Brainfuck-interpreter)


c++で書いたインタープリター。ネット上のサンプルプログラムを幾つか実行して思い通り動いたが、正しいかは不明。コンパイルして、実行時に brainfuck のファイルを引数で与えると実行します。


 [1]: http://ja.wikipedia.org/wiki/Brainfuck
