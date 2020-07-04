+++
date = "2016-08-19T20:38:21+09:00"
draft = false
title = "clang のSanitizerについて"
categories = ["Programing"]
+++

昔調べた時にclangで`-fsanitize=undefined`が使えなかったので、よく考えずに`address`を使っていた。新しいclangを入れようとしていろいろしていたら使えるようになったので、ついでにsanitizer周りについて調べてみた。

<!--more-->

## 環境

   sanitizerはどれが使えるかとかが結構環境(OS)依存なようなので重要

- Mac OS X El Capitan (10.11.6)
- clang 3.8.0 (`brew install llvm38  --with-asan --with-lldb`)

  `--with-lldb`は関係ないかもしれないけど、一応上のコマンドで入れた。`--with-asan`が重要

## 概要

clangで使えるsanitizerには[AddressSanitizer](http://clang.llvm.org/docs/AddressSanitizer.html),[ThreadSanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html),[MemorySanitizer](http://clang.llvm.org/docs/MemorySanitizer.html),[UndefinedBehaviorSanitizer](http://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)などがある。

コンパイル時に

~~~
clang++ -fsanitize=address hoo.cpp
~~~

のように`-fsanitize=hoge`とつけると、実行ファイルにsanitizerがリンクされて実行時の各種エラーを見つけてくれる。

Memory, Thread SAnitizerはMacをサポートしていないようだったのでAddressSanitizerとUndefinedBehaviorSanitizerについて書く。

## AddressSanitizer

Xcodeで入ってくるclangでも使えた。

メモリ関連のバグを実行時に検出してくれる。内容は[clangのドキュメント](http://clang.llvm.org/docs/AddressSanitizer.html)を見るのがわかりやすいが、ざっくりまとめると、


- 以下のようなエラーを検出する
  - head, stack, globalでの領域外アクセス
  - freeした領域の使用
  - return した後のstackの利用 (追加オプション)
  - Use-after-scope (追加オプション)
  - 多重free, 無効なfree
  - メモリリーク (experimental)
- 実行速度は2倍くらい遅くなる
- スタックメモリは3倍くらい使う

### 例

ありそうな配列外アクセス

~~~c++
// array-out-of-bound.cpp
int main() {
    int a[100];
    for(int i = 0; i <= 100; i++) a[i] = 100;
    return 0;
}
~~~

自分は使いこなせていないが、周辺のメモリの状態まで表示してくれる。

~~~sh
$ clang++-3.8 -fsanitize=address array-out-of-bound.cpp
$ ./a.out
=================================================================
==83804==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fff5cd683b0 at pc 0x000102e97e36 bp 0x7fff5cd681f0 sp 0x7fff5cd681e8
WRITE of size 4 at 0x7fff5cd683b0 thread T0
    #0 0x102e97e35 in main (a.out+0x100000e35)
    #1 0x7fff8a4965ac in start (libdyld.dylib+0x35ac)
    #2 0x0  (<unknown module>)

Address 0x7fff5cd683b0 is located in stack of thread T0 at offset 432 in frame
    #0 0x102e97cef in main (a.out+0x100000cef)

  This frame has 1 object(s):
    [32, 432) 'a' <== Memory access at offset 432 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow (a.out+0x100000e35) in main
Shadow bytes around the buggy address:
  0x1fffeb9ad020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad030: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad040: f1 f1 f1 f1 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad050: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad060: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x1fffeb9ad070: 00 00 00 00 00 00[f3]f3 f3 f3 f3 f3 f3 f3 f3 f3
  0x1fffeb9ad080: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad090: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad0a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad0b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb9ad0c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Heap right redzone:      fb
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack partial redzone:   f4
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==83804==ABORTING
[3]    83804 abort      ./a.out
~~~

また、`-g`をつけて、デバッグ情報付きでコンパイルすると以下のように、`a.out+hoge`だった部分がソースファイル名とエラーが出た行番号になる。

~~~
    #0 0x105f92e35 in main array-out-of-bound.cpp:8
    #1 0x7fff8a4965ac in start (libdyld.dylib+0x35ac)
    #2 0x0  (<unknown module>)

Address 0x7fff59c6d3b0 is located in stack of thread T0 at offset 432 in frame
    #0 0x105f92cef in main array-out-of-bound.cpp:6
~~~

## UndefinedBehaviorSanitizer

Xcodeで入ってきたclangでは使えなかった。`brew install llvm38 --with-asan --with-lldb`で入れたclangでは使えた。しかし、`--with-asan`はaddress sanitizerを入れるオプションなので、関連性は知らない。

実行中に様々な未定義動作を検出して教えてくれる。AddressSanitizerと違って、プログラムはcrashせず動き続ける。（オプションによる）

チェックしてくれる項目は[clangのドキュメント](http://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html#availablle-checks)参照。`-fsanitize=undefined`でunsigned-integer-overflow 以外が全て有効になる。


### 例1

さっきのプログラムに最後の出力を加えたもの

~~~c++
// array-out-of-bound.cpp
#include<iostream>
int main() {
    int a[100];
    for(int i = 0; i <= 100; i++) a[i] = 100;
    std::cout << "Finish" << std::endl;
    return 0;
}
~~~

エラーを表示しながらも、プログラムは動き続けている。

~~~sh
$ clang++-3.8 -fsanitize=undefined array-out-of-bound.cpp
$ ./a.out
array-out-of-bound.cpp:9:35: runtime error: index 100 out of bounds for type 'int [100]'
Finish
~~~


### 例2

オーバーフロー、アンダーフローの例

~~~c++
#include<climits>
int main() {
    int a = INT_MAX;
    a += 1;
    a -= 1;
    a -= INT_MAX;
    return 0;
}
~~~

嬉しい

~~~sh
$ clang++-3.8 -fsanitize=undefined integer-overflow.cpp
$ ./a.out
integer-overflow.cpp:8:7: runtime error: signed integer overflow: 2147483647 + 1 cannot be represented in type 'int'
integer-overflow.cpp:9:7: runtime error: signed integer overflow: -2147483648 - 1 cannot be represented in type 'int'
~~~

## 組み合わせる

AddressSanitizerとUndefinedBehaviorSanitizerは両方同時に使うことができる。

ただし、Address, Thread, Memoryはどれか１つしか同時に使えない。

~~~c++
// array-out-of-bound.cpp
#include<iostream>
int main() {
    int a[100];
    for(int i = 0; i <= 100; i++) a[i] = 100;
    std::cout << "Finish" << std::endl;
    return 0;
}
~~~

以下のように、未定義動作をruntime errorで表示しつつ、領域外アクセスなどがあったらabortして詳細を表示してくれる。

undefinedが結構強いので、addressはつけなくても十分かもしれない？

~~~sh
$ clang++-3.8 -fsanitize=undefined,address array-out-of-bound.cpp
$ ./a.out
array-out-of-bound.cpp:9:35: runtime error: index 100 out of bounds for type 'int [100]'
SUMMARY: AddressSanitizer: undefined-behavior array-out-of-bound.cpp:9:35 in
=================================================================
==19791==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fff5acce370 at pc 0x000104f31b20 bp 0x7fff5acce1b0 sp 0x7fff5acce1a8
WRITE of size 4 at 0x7fff5acce370 thread T0
    #0 0x104f31b1f in main (a.out+0x100000b1f)
    #1 0x7fff8a4965ac in start (libdyld.dylib+0x35ac)
    #2 0x0  (<unknown module>)

Address 0x7fff5acce370 is located in stack of thread T0 at offset 432 in frame
    #0 0x104f3194f in main (a.out+0x10000094f)

  This frame has 1 object(s):
    [32, 432) 'a' <== Memory access at offset 432 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow (a.out+0x100000b1f) in main
Shadow bytes around the buggy address:
  0x1fffeb599c10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb599c20: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb599c30: 00 00 00 00 00 00 00 00 f1 f1 f1 f1 00 00 00 00
  0x1fffeb599c40: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb599c50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x1fffeb599c60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00[f3]f3
  0x1fffeb599c70: f3 f3 f3 f3 f3 f3 f3 f3 00 00 00 00 00 00 00 00
  0x1fffeb599c80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb599c90: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb599ca0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1fffeb599cb0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Heap right redzone:      fb
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack partial redzone:   f4
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==19791==ABORTING
[3]    19791 abort      ./a.out
~~~


[1]: http://clang.llvm.org/docs/UsersManual.html#controlling-code-generation

