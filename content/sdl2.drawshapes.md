---
title: SDL2の使い方　②色々描く
author: cormoran
date: 2015-04-29
categories:
  - Programing
tags:
  - SDL2
---

[前回](blog/sdl2-showwindow/)の続きです。今回はSDL2のライブラリにある図形を描写する簡単なサンプルです。備忘録。
  
<!--more-->

## 背景色を変える & 直線描写

まずは直線だけの例。どのページからとってきたか忘れましたが[SDL Wiki][2]内のサンプルコードの改変です。

描写はRendererのキャンバスに色々書き込んでいってそれをWindowに転写するイメージだと思います。

~~~
//environment : gcc4.9 c++
#include "SDL2/SDL.h"

int main()
{
  SDL_Init(SDL_INIT_VIDEO);
  SDL_Window* window = SDL_CreateWindow("Draw Test",SDL_WINDOWPOS_CENTERED,SDL_WINDOWPOS_CENTERED,640,480,0);
  SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, 0);

  SDL_Event ev;
  while(true){

    //描写色を変える
    //R,G,B,A)
    //注：AはSDL_SetRenderDrawBlendModeの設定をしないと無効
    SDL_SetRenderDrawColor(renderer, 0, 255, 200, 255);

    //ウィンドウを現在の色で塗りつぶす　つまり消去
    SDL_RenderClear(renderer);
    
    //ウィンドウなどのイベント処理
    //ifにするとeventがキューにどんどん溜まってしまって反応が遅くなる？
    //whileでもMacだと固まることがあってよく分からない
    while (SDL_PollEvent(&ev)) { 
      if (ev.type == SDL_QUIT) {
      	break;
       }
    } 

    //描写する色を切り替える
    SDL_SetRenderDrawColor(renderer, 200, 150, 150, 255);

    //直線を描く
    //x1,y1,x2,y2の順
    SDL_RenderDrawLine(renderer,10, 10, 50, 50);
    
    //windowにレンダリングする
    SDL_RenderPresent(renderer);
        
  }

  //解放処理
  SDL_DestroyRenderer(renderer);
  SDL_DestroyWindow(window);
  SDL_Quit();

  return 0;
  
}
~~~

## 色々描写する

SDLには円を描写する関数がないみたいです。点、直線、長方形を描写する関数があります。一通り使ってみました。

SDL2_GFXを使うと円などを書く関数も使えるようです。

~~~
 //environment : gcc4.9 c++
#include "SDL2/SDL.h"

int main(int, char ** const)
{
  SDL_Init(SDL_INIT_VIDEO);
  SDL_Window* window = SDL_CreateWindow("Draw Test",SDL_WINDOWPOS_CENTERED,SDL_WINDOWPOS_CENTERED,640,480,0);
  SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, 0);

  
  SDL_Event ev;
  while(true){

    //描写色を変える
    //R,G,B,A)
    //AはDrawBlendModeの設定が必要らしい
    SDL_SetRenderDrawColor(renderer, 0, 255, 200, 255);

    //ウィンドウを現在の色で塗りつぶす
    SDL_RenderClear(renderer);
    
    while (SDL_PollEvent(&ev)) {
      if (ev.type == SDL_QUIT) {
	break;
      }
    }
    
    //描写色切り替え
    SDL_SetRenderDrawColor(renderer, 200, 150, 150, 255);

    //直線を描く
    //x1,y1,x2,y2
    SDL_RenderDrawLine(renderer,10, 10, 50, 50);
    for(int x=60;x&lt;=100;x+=2){
      for(int y=10;y&lt;=50;y+=2){
	//点を描く
	SDL_RenderDrawPoint(renderer,x,y);
      }
    }
    //下の長方形の例のように配列でやることも可能


    //長方形（枠だけ）を描く
    SDL_Rect rect1;
    rect1.x=110;rect1.y=10;rect1.w=100;rect1.h=100;
    SDL_RenderDrawRect(renderer, &rect1);


    //長方形（塗りつぶし）を描く
    SDL_Rect rect2=(SDL_Rect){220,10,100,100};
    SDL_RenderFillRect(renderer,&rect2);


    //長方形を配列を使ってまとめて描く
    SDL_Rect rects[20];
    for(int i=0;i&lt;20;i++){
      rects[i]=(SDL_Rect){10+i*20,150,10,10};
    }
    SDL_RenderDrawRects(renderer, rects,20);


    
    //windowにレンダリングする
    SDL_RenderPresent(renderer);
        
  }

  SDL_DestroyRenderer(renderer);
  SDL_DestroyWindow(window);

  SDL_Quit();

  return 0;
  
}
~~~

 [2]: https://wiki.libsdl.org/FrontPage
 