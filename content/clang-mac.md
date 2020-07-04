+++
date = "2016-08-19T20:28:01+09:00"
draft = false
title = "Macのclangでlibstdc++,libc++を切り替えて使う"
categories = ["Programing"]
+++

Mac OS 10.8まで(?)は、Xcodeで入ってくるclangはlibstdc++を使っていたが、最近はlibc++を使うように変更されたらしい。

Macで、**clangで**libstdc++を使いたくなったのでやってみた。

標準ライブラリの切り替えとかは険しそうな雰囲気があるけど、結構簡単にできることがわかった。（あまり動作確認していないのでうまくいってるのかはわからない）

<!--more-->

よくわかってない部分が多々あるので注意。

## 環境

- Mac OS X El Capitan (10.11.6)
- clang 3.8.0 (brew install llvm38)
- gcc 5.3.0 (brew install gcc)


どのライブラリのヘッダをincludeしているかは以下のコードで調べた。

~~~c++
// check.cpp
#include<cstdio>

int main() {
#ifdef _LIBCPP_VERSION
    std::printf("libc++: %d\n", _LIBCPP_VERSION);
#endif

#ifdef __GLIBCXX__
    std::printf("GLIBCXX: %d\n", __GLIBCXX__);
#endif
    return 0;
}
~~~

どのライブラリをリンクしているかは`otool`で調べた。


## libc++ を使う

普通に`clang-38 check.cpp`してもlibc++が使われるが、`brew info llvm38`曰く

~~~
To link to libc++, something like the following is required:
  CXX="clang++-3.8 -stdlib=libc++"
  CXXFLAGS="$CXXFLAGS -nostdinc++ -I/usr/local/opt/llvm38/lib/llvm-3.8/include/c++/v1"
  LDFLAGS="$LDFLAGS -L/usr/local/opt/llvm38/lib/llvm-3.8/lib"
~~~

らしいのでつけておく。

~~~sh
$ alias clang++-3.8-with-libcpp
clang++-3.8-with-libcpp='clang++-3.8 -stdlib=libc++ -nostdinc++ -I/usr/local/opt/llvm38/lib/llvm-3.8/include/c++/v1 -L/usr/local/opt/llvm38/lib/llvm-3.8/lib '

$ clang++-3.8-with-libcpp -v check.cpp
clang version 3.8.0 (tags/RELEASE_380/final)
Target: x86_64-apple-darwin15.6.0
Thread model: posix
InstalledDir: /usr/local/bin
 "/usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/clang" -cc1 -triple x86_64-apple-macosx10.11.0 -Wdeprecated-objc-isa-usage -Werror=deprecated-objc-isa-usage -emit-obj -mrelax-all -disable-free -main-file-name check.cpp -mrelocation-model pic -pic-level 2 -mthread-model posix -mdisable-fp-elim -masm-verbose -munwind-tables -target-cpu core2 -target-linker-version 264.3.102 -v -dwarf-column-info -debugger-tuning=lldb -nostdinc++ -resource-dir /usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/../lib/clang/3.8.0 -I /usr/local/opt/llvm38/lib/llvm-3.8/include/c++/v1 -stdlib=libc++ -fdeprecated-macro -fdebug-compilation-dir / -ferror-limit 19 -fmessage-length 181 -stack-protector 1 -fblocks -fobjc-runtime=macosx-10.11.0 -fencode-extended-block-signature -fcxx-exceptions -fexceptions -fmax-type-align=16 -fdiagnostics-show-option -fcolor-diagnostics -o /var/folders/h9/4dy9093x2sncnxjnrwwqn5vc0000gn/T/check-ea6fd2.o -x c++ check.cpp
 clang -cc1 version 3.8.0 based upon LLVM 3.8.0 default target x86_64-apple-darwin15.6.0
 #include "..." search starts here:
 #include <...> search starts here:
  /usr/local/opt/llvm38/lib/llvm-3.8/include/c++/v1
  /usr/local/include
  /usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/../lib/clang/3.8.0/include
  /usr/include
  /System/Library/Frameworks (framework directory)
  /Library/Frameworks (framework directory)
 End of search list.
 "/usr/bin/ld" -demangle -dynamic -arch x86_64 -macosx_version_min 10.11.0 -o a.out -L/usr/local/opt/llvm38/lib/llvm-3.8/lib /var/folders/h9/4dy9093x2sncnxjnrwwqn5vc0000gn/T/check-ea6fd2.o -lc++ -lSystem /usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/../lib/clang/3.8.0/lib/darwin/libclang_rt.osx.a

$ ./a.out
libc++: 3800

$ otool -L a.out
a.out:
        /usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 120.1.0)
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)'

~~~

ちょっといいのかわからない点(後述)があるが、とりあえずlibc++を使っている。

## libstdc++ を使う

`brew info llvm38`で言われたことを頼りにgcc-5についてきたlibstdc++をclang-38で使ってみる。

~~~
-stdlib=libstdc++ \
-nostdinc++ \
-I/path/to/include/ \
-L/path/to/lib/
~~~

上のような感じでオプションをつければうまくいくかもしれない

`g++-5 -v check.cpp` した結果以下のようなところを追加すればよさそうだった（結構適当）

~~~
# IncludeSearchPath
/usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0
/usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/x86_64-apple-darwin15.0.0
/usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/backward
# エラーになるので消す
# /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include
# LibrarySearchPath
/usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0
/usr/local/Cellar/gcc/5.3.0/lib/gcc/5
~~~

やってみるとちゃんとlibstdc++がリンクされている。簡単なプログラムをいくつかコンパイルしてみたが、問題なく動いていた。

<ins>追記</ins>

algorithmのライブラリを使ったら`__builtin`系の関数がないよと言ってエラーになった。`/usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include`にあるヘッダを読んでエラーになっていたので、include pathから消してみると動いた。他にもエラーが起きる部分があるかもしれない。


~~~sh
$ alias clang++-3.8-with-libstdc++
clang++-3.8-with-libstdc++='clang++-3.8 -stdlib=libstdc++ -nostdinc++ -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0 -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/x86_64-apple-darwin15.0.0 -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/backward -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include -isystem /usr/local/Cellar/gcc/5.3.0/include -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include-fixed -L /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0 -L /usr/local/Cellar/gcc/5.3.0/lib/gcc/5 '

$ clang++-3.8-with-libstdc++ -v check.cpp
clang version 3.8.0 (tags/RELEASE_380/final)
Target: x86_64-apple-darwin15.6.0
Thread model: posix
InstalledDir: /usr/local/bin
 "/usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/clang" -cc1 -triple x86_64-apple-macosx10.11.0 -Wdeprecated-objc-isa-usage -Werror=deprecated-objc-isa-usage -emit-obj -mrelax-all -disable-free -main-file-name check.cpp -mrelocation-model pic -pic-level 2 -mthread-model posix -mdisable-fp-elim -masm-verbose -munwind-tables -target-cpu core2 -target-linker-version 264.3.102 -v -dwarf-column-info -debugger-tuning=lldb -nostdinc++ -resource-dir /usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/../lib/clang/3.8.0 -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0 -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/x86_64-apple-darwin15.0.0 -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/backward -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include -isystem /usr/local/Cellar/gcc/5.3.0/include -isystem /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include-fixed -stdlib=libstdc++ -fdeprecated-macro -fdebug-compilation-dir / -ferror-limit 19 -fmessage-length 181 -stack-protector 1 -fblocks -fobjc-runtime=macosx-10.11.0 -fencode-extended-block-signature -fcxx-exceptions -fexceptions -fmax-type-align=16 -fdiagnostics-show-option -fcolor-diagnostics -o /var/folders/h9/4dy9093x2sncnxjnrwwqn5vc0000gn/T/check-9b0204.o -x c++ check.cpp
 clang -cc1 version 3.8.0 based upon LLVM 3.8.0 default target x86_64-apple-darwin15.6.0
 #include "..." search starts here:
 #include <...> search starts here:
  /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0
  /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/x86_64-apple-darwin15.0.0
  /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/../../../../../../include/c++/5.3.0/backward
  /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include
  /usr/local/Cellar/gcc/5.3.0/include
  /usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0/include-fixed
  /usr/local/include
  /usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/../lib/clang/3.8.0/include
  /usr/include
  /System/Library/Frameworks (framework directory)
  /Library/Frameworks (framework directory)
End of search list.
"/usr/bin/ld" -demangle -dynamic -arch x86_64 -macosx_version_min 10.11.0 -o a.out -L/usr/local/Cellar/gcc/5.3.0/lib/gcc/5/gcc/x86_64-apple-darwin15.0.0/5.3.0 -L/usr/local/Cellar/gcc/5.3.0/lib/gcc/5 /var/folders/h9/4dy9093x2sncnxjnrwwqn5vc0000gn/T/check-9b0204.o -lstdc++ -lSystem /usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/bin/../lib/clang/3.8.0/lib/darwin/libclang_rt.osx.a
$ ./a.out
GLIBCXX: 20151204

$ otool -L a.out
a.out:
	/usr/local/opt/gcc/lib/gcc/5/libstdc++.6.dylib (compatibility version 7.0.0, current version 7.21.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)'
~~~


## よくわからない点

libc++でリンクされていたライブラリは`/usr/lib/libc++.1.dylib`で、これはパス的にも更新日的にもXcodeかOSかが入れたやつな気がする。

llvm38のlibc++を探すと見つかった。

~~~
$ find /usr/local/Cellar/llvm38 -name libc++*
/usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/usr/lib/libc++.1.dylib
/usr/local/Cellar/llvm38/3.8.0/lib/llvm-3.8/usr/lib/libc++.dylib
~~~


こっちが使われるべきな気がするけど使わせる方法がよくわからなかった。libc++のサイトを調べて-Wl,rpathとか加えてみたりいろいろしたけど今回は諦める。

## 追記

色々エラーが出る場面（詳しくは忘れた）に遭遇したのでやめた