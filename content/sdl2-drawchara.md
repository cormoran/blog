---
title: SDL2の使い方　③文字を表示
author: cormoran

date: 2015-04-30

categories:
  - Programing
tags:
  - SDL2

---

またまた続きです。今回はSDL2_TTFを使用して文字を表示します。

<!--more-->

SDL2 TTFを使うと、true type フォントを使って文字を描くことができます。

true typeのフォントは自分でどこかからとってきてfont.ttfという名前で保存してください。自分は普段使っているRictyというフォントで試しましたが、日本語もちゃんと表示してくれました。文字コードはUTF8を想定して書いていますが何種類か設定はあったと思います。Rictyはライセンスの関係で自分でコンパイルして作らないといけないので[RictyDiminished](https://github.com/yascentur/RictyDiminished)がサクッと使うにはおすすめです。（プログラミングのエディタ用フォントなのでゲームとかには別のフォントを使ったほうが良い）

コンパイルはSDL2の時のオプションに加えて　-lSDL2_ttf　を付けてライブラリをリンクする必要があります。

~~~
  
#include <SDL2/SDL.h>  
#include <SDL2/SDL_ttf.h>

//使うフォントを実行ファイルのディレクトリに入れておく
  
#define FONT_PATH "font.ttf"

int main(int, char ** const) {
    
    SDL_Init(SDL_INIT_VIDEO);
    TTF_Init();
    
    SDL_Window* window = SDL_CreateWindow("Draw chara Test",SDL_WINDOWPOS_CENTERED,SDL_WINDOWPOS_CENTERED,640,480,0);
    SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, 0);
    SDL_Surface *surface;
    SDL_Texture *texture;
    SDL_Event ev;

    TTF_Font *font;

    font=TTF_OpenFont(FONT_PATH, 50);

    //TTF_SetFontOutline(font, 1);//枠抜きで描写するとき
    
    surface = TTF_RenderUTF8_Blended(font, "HelloWorld!", (SDL_Color){255,255,255,255});
    
    //surfaceからTextureを作る?    
    texture =SDL_CreateTextureFromSurface(renderer, surface);

    while(true){
      
        SDL_SetRenderDrawColor(renderer, 0, 255, 200, 255);      
        SDL_RenderClear(renderer);

        while (SDL_PollEvent(&ev)) {        
            if (ev.type == SDL_QUIT) {	  
                break;        
            }      
        }

        //文字を描写したTextureのサイズを取得する      
        int iw,ih;      
        SDL_QueryTexture(texture, NULL, NULL, &iw, &ih);
      
        SDL_Rect txtRect=(SDL_Rect){0,0,iw,ih};      
        SDL_Rect pasteRect=(SDL_Rect){100,100,iw,ih};

        //Textureを描写する      
        //描写元の描写する部分,描写先の描写する部分)      
        //サイズが違うと勝手にTextureを伸展してくれる      
        SDL_RenderCopy(renderer, texture, &txtRect, &pasteRect);

        //windowにレンダリングする      
        SDL_RenderPresent(renderer);
    
    }

    SDL_FreeSurface(surface);
    SDL_DestroyTexture(texture);
    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();
    TTF_CloseFont(font);
    TTF_Quit();
    
    return 0;

}

~~~
