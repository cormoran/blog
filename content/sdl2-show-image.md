---
title: SDL2の使い方　④画像を表示
author: cormoran
date: 2015-06-08
categories:
  - Programing
tags:
  - SDL2
---

SDL2_imageを使って画像を表示するサンプルプログラム。
  
<!--more-->
  
とりあえず備忘録的に置いておく。変なところがあるかもしれない。

## コンパイル

~~~
gcc `sdl2-config --cflags --libs` -lsdl2_image foo.c
~~~


## プログラム

* 実行ファイルと同じフォルダにimage1.pngを置いておく。

* 自分の環境ではlibpngのwarningが出た。まだ調べていない。
  
~~~
  
#include <SDL.h>
  
#include <SDL_image.h>

int main(int, char ** const)
  
{
    
    SDL_Init(SDL_INIT_VIDEO);
    
    SDL_Window* window = SDL_CreateWindow(Draw Test,SDL_WINDOWPOS_CENTERED,SDL_WINDOWPOS_CENTERED,640,480,0);
    
    SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, 0);

    //SDL_Image初期化
    
    int flags = IMG_INIT_PNG;//使う画像形式を指定
    
    int initted = IMG_Init(flags);
    
    if((initted & flags) != flags) {
        printf("IMG_Init: Failed to init required jpg and png support!\n");
        printf("IMG_Init: %s\n", IMG_GetError());
    
    }
    
    //画像をロード
    SDL_Surface *image = IMG_Load(image1.png);
    
    if(!image){
        printf("IMG_Load: %s\n", IMG_GetError());
    }
    
    SDL_Texture *image_texture;
    image_texture = SDL_CreateTextureFromSurface(renderer, image);
    SDL_Event ev;
    
    while(true){

        SDL_SetRenderDrawColor(renderer, 200, 200, 200, 255);
      
        //ウィンドウを現在の色で塗りつぶす　つまり消去
        SDL_RenderClear(renderer);

        //ここwhileにすべきと思うが自分のMacではwhileにするとうまくQuitを取れなかった
        if (SDL_PollEvent(&ev)) {
            if (ev.type == SDL_QUIT) {
                break;
            }
        }

        int iw,ih;
      
        //画像テクスチャのサイズを取得
      
        SDL_QueryTexture(image_texture, NULL, NULL, &iw, &ih);
      
        SDL_Rect image_rect = (SDL_Rect){0,0,iw,ih};
        SDL_Rect draw_rect = (SDL_Rect){50,50,iw,ih};
        SDL_RenderCopy(renderer,image_texture, &image_rect, &draw_rect);

        //windowにレンダリングする
      
        SDL_RenderPresent(renderer);
      
        SDL_Delay(10);
    
    }

    IMG_Quit();
    SDL_FreeSurface(image);
    SDL_DestroyTexture(image_texture);
    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();

    return 0;

}

~~~
