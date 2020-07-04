+++
date = "2016-08-20T02:55:10+09:00"
draft = false
title = "libc++とlibstdc++のデバッグモードについて"
categories = ["Programing"]
+++

 `-D_GLIBCXX_DEBUG` をつけていたけど、実は効果がない状態だった...　という話。

<!--more-->

libc++では`-D_GLIBCXX_DEBUG`してもデバッグモードにならない。

`-D_LIBCPP_DEBUG=0`とつける必要がある。

しかし、開発中で対応しているライブラリはまだ少ない様子。

## libstdc++ のデバッグモード

[ドキュメントはここ](https://gcc.gnu.org/onlinedocs/libstdc++/manual/debug_mode_using.html#debug_mode.using.mode)

libstdc++では、`_GLIBCXX_DEBUG`を定義しておくと標準ライブラリにエラーチェックなどが追加される。vector, string, map, set, list, dequeなどは定義関係なくエラーチェックを入れたライブラリを別に用意してくれているほどなので、少なくともこれらのコンテナではエラーチェックが効くはず。

デバッグモードになると、Iteratorのエラー（配列外アクセスなど）検出や、アルゴリズムの前提条件チェック（ソートしてるかなど）をしてくれるらしい。


### 例１ vectorの配列外アクセス

~~~c++
// vector-over.cpp
#include<iostream>
#include<vector>
using namespace std;
int main() {
    vector<int> a(100);    
    for(int i = 0; i <= 100; i++) a[i] = 100;
    std::cout << "Finish" << std::endl;
    return 0;
}
~~~

~~~sh
$ g++-5 -D_GLIBCXX_DEBUG vector-over.cpp
$ ./a.out
/usr/local/Cellar/gcc/5.3.0/include/c++/5.3.0/debug/vector:406:error:
    attempt to subscript container with out-of-bounds index 100, but
    container only holds 100 elements.

Objects involved in the operation:
sequence "this" @ 0x0x7fff50244440 {
  type = NSt7__debug6vectorIiSaIiEEE;
}
[3]    61209 abort      ./a.out
~~~

### 例２ 配列がソートされていることを前提にしたアルゴリズム

~~~c++
// inter-section.cpp
// ref: http://www.cplusplus.com/reference/algorithm/set_intersection/
#include <iostream>
#include <algorithm>
#include <vector>

int main () {
  int first[] = {5,10,15,20,25};
  int second[] = {50,40,30,20,10};
  std::vector<int> v(10);                      // 0  0  0  0  0  0  0  0  0  0
  std::vector<int>::iterator it;

  //  std::sort (first,first+5);     //  5 10 15 20 25
  //  std::sort (second,second+5);   // 10 20 30 40 50

  it=std::set_intersection (first, first+5, second, second+5, v.begin());
                                               // 10 20 0  0  0  0  0  0  0  0
  v.resize(it-v.begin());                      // 10 20

  std::cout << "The intersection has " << (v.size()) << " elements:\n";
  for (it=v.begin(); it!=v.end(); ++it)
    std::cout << ' ' << *it;
  std::cout << '\n';

  return 0;
}
~~~

ソートされてないよと教えてくれる

~~~sh
$ g++-5 -D_GLIBCXX_DEBUG inter-section.cpp
$ ./a.out
/usr/local/Cellar/gcc/5.3.0/include/c++/5.3.0/bits/stl_algo.h:5120:error:
    elements in iterator range [__first2, __last2) are not sorted.

Objects involved in the operation:
iterator "__first2" @ 0x0x7fff5f580860 {
type = Pi;
}
iterator "__last2" @ 0x0x7fff5f580858 {
type = Pi;
}
[3]    5262 abort      ./a.out
~~~

## libc++ のデバッグモード

[ドキュメントはここ](http://libcxx.llvm.org/debug_mode.html)

あまり情報を得られなかったが、デバッグモードは開発中らしい。

`-D_LIBCPP_DEBUG=0`をつけるとvectorの配列外アクセスは検出してくれた。

`-D_LIBCPP_DEBUG=1`とすると、さらに別のコードが増えるらしく、コンパイルエラーになった。

対応したライブラリが少なく、実行時のエラー表示もlibstdc++の方が良い感じだが、開発が進んだらこっちも良くなるのかもしれない。

libstdc++で試したのと同じプログラムでどうなるか見てみた。

### 例１ vectorの配列外アクセス

プログラムはさっきと同じ

~~~sh
$ clang++-3.8 -D_LIBCPP_DEBUG=0 vector-over.cpp
$ ./a.out
vector[] index out of bounds
[3]    13859 abort      ./a.out
~~~

### 例２ 配列がソートされていることを前提にしたアルゴリズム

未対応っぽい。特に何も言われない

~~~
$ clang++-3.8 -D_LIBCPP_DEBUG=0 inter-section.cpp
$ ./a.out
The intersection has 0 elements:

~~~

## 感想

libstdc++は色々なところでデバッグモードが実装されてそうなのでつけておくと助けられるかもしれない。

ただ、vectorの配列外アクセスなどは、ソースコードのどこでやってしまったのかわからないのでsanitizer使った方が良さそうな気もする。








## 参考

- [The GNU C++ Library Manual][1]
- ["libc++" C++ Standard Library][2]
- [C++標準ライブラリをdebug modeで使う](http://qiita.com/tell/items/28fa6e76003aecb908a3)
- [twitterのお話](https://twitter.com/natrium11321/status/515366162633203712)

[1]: https://gcc.gnu.org/onlinedocs/libstdc++/manual/
[2]: http://libcxx.llvm.org/

