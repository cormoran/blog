---
title: SDL2の使い方　①ウィンドウの表示
author: cormoran
date: 2015-04-28
categories:
  - Programing
tags:
  - SDL2
---

訳あってSDL2を使うことになったので色々やってみました。自分の備忘録とこれからSDL2を使う人のためにシンプルなサンプルを残しておきたいと思います。今回は基本のウィンドウ表示です。
  
<!--more-->

## インストール(Macの場合)

sdl2本体以外にも、いろいろ使えるライブラリがあるので一緒に入れておくと良いです。

~~~
$ brew install sdl2 sdl2_image sdl2_ttf sdl2_mixer sdl2_gfx sdl2_net
~~~

## コンパイル

コンパイルはSDL2のインストールの仕方にもよりますが、Mac環境、brewでインストールした場合sdl2-configにパスを勝手に設定してくれるので、

~~~
  
gcc source.c `sdl2-config --cflags --libs`
  
~~~
  
でいけると思います。

sdl2-config --cflags --libs でSDL2で必要なcflg等を勝手に追加してくれます。

~~~

sdl2-config --cflags --libs

~~~

とすると、gccのオプション -I や -Lが出てきて、何をやっているかよくわかります。

## Window表示

SDL2のマニュアルは[SDL Wiki][1]にあり、このページにいくつかサンプルもあります。
かなり詳しく書かれているので正直これを読めば大体分かります。
以下のコードはコメント以外はサンプルほとんどそのままです。

~~~
  
//This code is from
 
//https://wiki.libsdl.org/SDL_CreateWindow?highlight=%28%5CbCategoryVideo%5Cb%29%7C%28CategoryEnum%29%7C%28CategoryStruct%29

#include <SDL2/SDL.h>//ここはパスの通し方等で少し変えないといけないかもしれない  
#include <cstdio>

int main() {

    SDL_Init(SDL_INIT_VIDEO);//SDL2の初期化

    SDL_Window *window = SDL_CreateWindow(
        SDL2 window, //ウィンドウタイトル
        SDL_WINDOWPOS_UNDEFINED, //ウィンドウ位置x
        SDL_WINDOWPOS_UNDEFINED, //y
        640, //幅
        480, //高さ
        SDL_WINDOW_OPENGL //flags 詳細はマニュアル参照
	);

    if (window == NULL) {
        printf("Could not create window: %s\n", SDL_GetError());
        return 1;
    }

    SDL_Delay(5000); //delay ms
    SDL_DestroyWindow(window);
    SDL_Quit();
    return 0;
}

~~~

 [1]: https://wiki.libsdl.org/FrontPage
 
