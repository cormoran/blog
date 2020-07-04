+++
date = "2016-08-19T17:25:59+09:00"
draft = false
title = "clang の警告オプションについて"
categories = ["Programing"]
+++

今までとりあえず`-Wall`をつけていたけど、他にも役立つ警告がいろいろあった。

<!--more-->

## 環境

- Mac OS X El Capitan (10.11.6)
- clang 3.8.0 (brew install llvm38 で インストール)

## 主な警告オプション一覧

clangのコマンドでもどうにかすれば見られるのかもしれないが、よくわからない。ドキュメントも見たが、詳細は見つからなかったので適当にWebから探した。

[ここの階層構造][2]と[ここの一覧][3]を対応させて見るのがわかりやすかった。また、zshの補完で説明も含めて全部出てきたのでそれも参考にした。

### `-W`

This switch is deprecated; use -Wextra instead らしい
効果は-Wextraと同じ

### `-Wall`

allとか言ってるから全部のように見えるけど、clangが持ってる警告オプションの半分くらいしか有効にならない。gccのWallの定義に合わせたとか。[ここ][1]に詳しい話が載っている。

### `-Wextra`

Wallを含んでいるのだ思っていたがそうではなかった

Wsign-compare, Wunused-parameter などが有効になる。これらはWallには含まれていない。

### `-Weverything`

（本当に）全ての警告オプションを有効にする。これが一番強い。

### `-Werror`

警告をエラーとして扱う

### `-Wno-hoge`

hogeの警告を無効にする



## Wallでは有効にならない、いい感じのオプション

`-Weverything`でコンパイルするとsize_tとintとかを全部警告してきてうるさいのだが、かなり使えそうな警告もあった。

### `-Wfloat-equal`

浮動小数点を==で比較するのを警告してくれる

### `-Wshadow`

ローカル変数の名前が外のスコープの変数とかぶっていたら警告してくれる。

下のようにループ変数名の重複を警告してくれるのでとても良さそう。

~~~c++
for(int i = 0; i < 10; i++) {
    for(int j = 0; j < 10; j++) {
        for(int i = 0; i < 10; i++) {
                
        }
    }
}
~~~

~~~sh
clang++ a.cpp -Weverything
~~~

~~~sh
a.cpp:15:21: warning: declaration shadows a local variable [-Wshadow]
for(int i = 0; i < 10; i++) {
        ^
a.cpp:13:13: note: previous declaration is here
for(int i = 0; i < 10; i++) {
        ^
1 warning generated.
~~~

## 使うことにした警告オプション

とりあえず`-Weverything`で全部のせにして、使っていく中で邪魔なものを消していくことにした。

既存のコードをいくつかコンパイルしてみて現状以下のような感じになった。

競技プログラミングで使うという観点から選んだので、開発等で使うなら`-Wno`しないほうが良さそうなものもいくつか無効にしている。

~~~sh
clang++ -std=c++14 \
-Weverything \
-Wno-old-style-cast \
-Wno-missing-prototypes \
-Wno-c++98-compat-pedantic \
-Wno-exit-time-destructors \
-Wno-global-constructors \
-Wno-sign-conversion \
-Wno-missing-variable-declarations \
-Wno-unused-macros \
-Wno-unused-const-variable
~~~

無効化したオプションの簡単な説明

| オプション                          | 説明 |
|:------------------------------------|:-----|
|`-Wno-old-style-cast`                | (int) みたいなキャスト|
|`-Wno-missing-prototypes`            | プロトタイプ宣言がない関数 |
|`-Wno-c++98-compat-pedantic`         | c++98互換でない（vector<vector<int>> みたいな）もの |
|`-Wno-exit-time-destructors`         | デストラクタを持つグローバル変数みたいなもの（？）
|`-Wno-global-constructors`           | コンストラクタを持つグローバル変数（？）
|`-Wno-sign-conversion`               | singed, unsigned 間の非明示的な変換　配列の添字にint使った時とかに怒られる
|`-Wno-missing-variable-declarations` | extern 宣言のないグローバル変数（？）
|`-Wno-unused-macros`                 | 使用されていないマクロ
|`-Wno-unused-const-variable`         | 使用されていないconst変数

	
## 参考

- [Better Apps with Clang's Weverything or Wall is a Lie!][1]
- [clang warning flags][2]
- [Which Clang Warning Is Generating This Message?][3]

[1]:http://amattn.com/p/better_apps_clang_weverything_or_wall_is_a_lie.html

[2]:https://gist.github.com/d0k/3608547

[3]:http://fuckingclangwarnings.com

