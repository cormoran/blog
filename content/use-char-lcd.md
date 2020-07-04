---
title: ＬＣＤキャラクタディスプレイを使ってみた
author: cormoran

date: 2014-09-07
categories:
  - 電子工作

---

![Hello World](https://lh3.googleusercontent.com/-M84jfbkPObE/ViUJvNdUBJI/AAAAAAAAADg/VaFecg_taGQ/s640-Ic42/lcd.helloworld.jpg)

<!--more-->

## 概要

* LCDに何か表示するテストプログラム。

* 今後再利用しやすくするためにライブラリ化した。

## 使用部品

* SC1602BS*B

* PIC16F1827

* 可変抵抗10KΩ

## 回路図

![回路図](https://lh3.googleusercontent.com/-yFnSi7rdWeI/ViUIor80BAI/AAAAAAAAABw/7G6bKjSSD70/s800-Ic42/SC1602BSBTest.png)

## プログラム

[GitHub AVR](https://github.com/cormoran/AVR)

今回作ったlcd表示用ライブラリです。下に使用例のプログラムがあります。

## ライブラリの設定

以下は備忘録です（つまり参考にはならない）

設定はlcd.hの ### で囲まれた部分にまとめています。ただし無理やり設定可能にしたので設定変更後の動作は未確認です。

_XTAL_FREQはdelay関数を使うために必要らしいです。システムクロックを指定します。

ポート設定で好きなピンを指定することができます。このプログラムではRS,RW,Eは自由に設定でき、DBはPortA/Bなどの連続4ポート弟子か設定できないようになっています。ポートの入出力設定(TRIS)はDBで使うポートのみライブラリ内で行っており、RS,RW,Eは自分で出力設定にする必要があります。

DB_TRIS_lcd , DB_lcd PORTA は使用するポート(A/Bなど)を指定、TRIS_mask_lcd はDBとして使用するピンのピット部分が1になるように設定します。

DB_ReadData_lcd、DB_SetData_lcdは特に無理矢理感が強いですが

DB_ReadData_lcd : DBの値を読み取ってそのデータが上位４ビットを占めるようにビットシフトさせる

DB_SetData_lcd : 上位4ビットに有効データが含まれる1byte変数 data を受け取りDBから出力する。有効データ部分がDBに行くようにビットシフトさせる

というような感じです。

//#define USE_BUSYFLG　の部分はコメントアウトを外すとBusyFlgを読み取ることで待機するはずなのですがやってみても動作しませんでした。lcd_ready関数にバグがあるのだと思いますが現在はdelay関数で代替して動いているので放置してあります。実は上で書いたDB_ReadData_lcdは現在は使っていないので設定しなくても動きます。

最後の void putchはprintf関数を使うための関数です。別の目的でputchを作る場合はlcd.cの#include,putch関数 部分とともにコメントアウトしておいてください。

~~~

//lcd.h 設定部分抜粋

/\*クロック設定\*/

#define _XTAL_FREQ 8000000 // delay関数のために必要　システムクロックを設定

/\*ポート設定\*/

#define RS_lcd RA4

#define RW_lcd RA6

#define E_lcd RA7

#define DB_TRIS_lcd TRISA //4ポートを使用

#define DB_lcd PORTA

#define TRIS_mask_lcd 0x0F //例: 0xF0でR?4/5/6/7 , 0x0FでR?0/1/2/3がDBポートとして使われる （?=A,Bなど）

#define DB_ReadData_lcd ((DB_lcd & TRIS_mask_lcd)<<4) //上位4bitにデータが入り下位4bitは0000となるように調節する 4/5/6/7ポートなら<<0、0/1/2/3ポートなら<>4 ) | (DB_lcd & ~TRIS_mask_lcd) ) //上位4bitに送信データが入っているのでそれがDBポートに出るように調節する4/5/6/7ポートなら>>0、0/1/2/3ポートなら>>4

/\*その他設定\*/

//#define USE_BUSYFLG //動作しないのでコメントアウト必須

void putch(char data);

~~~

## ライブラリの使い方

1.初期化

・lcd_init() (ここでライブラリ内で設定したDBポートは入出力が設定されます。)

・RS,RW,Eに使うポートを出力に設定

2.文字の表示など

lcd.cの制御用関数の部分にほとんどはまとめています。説明はlcd.cの各部文参照。

~~~
lcd_ShowString(unsigned char* strdata)

lcd_send_data(unsigned char data)

lcd_MoveLine(unsigned char line)

lcd_ClearDisplay(void)

lcd_CursorAtHome(void)

lcd_DisplayShift_L(int n)

lcd_DisplayShift_R(int n)

lcd_CursorShift_L(int n)

lcd_CursorShift_R(int n)

lcd_Set_DisplayPosition(char address)
~~~

上の関数に加えstdio.hをインクルードすればprintfで出力も可能です。数値などを出力する場合は

```
printf(&#8220;X=%d&#8221;,<変数>);
```

などとおなじみの形式で楽に出力できます。ただしprintfはプログラムが大きいらしいので（確認はしていない）メモリ不足の場合は注意が必要。

サンプルプログラム

~~~

/\* LCD SC1602BS\*B Test For PIC16F1827

* File: main.c

* Author: cormoran

*

* Created on 2014/09/05

*/

#define _XTAL_FREQ 8000000 // delay_ms(x) のための定義

#include

#include //これを書くとprintfが使える

//ライブラリ内でputch関数を定義しているので別の用途で使う場合は注意

#include&#8221;lcd.h&#8221; //lcdライブラリを使うために必要

#pragma config CLKOUTEN=OFF , FOSC=INTOSC , FCMEN=OFF , IESO=OFF , BOREN=ON , PWRTE=ON , WDTE=OFF , MCLRE=OFF , CP=OFF , CPD=OFF

#pragma config PLLEN=OFF , STVREN=ON , WRT=OFF , BORV=HI , LVP=OFF

void init(void){

OSCCON=0b01110010 ; //内部クロック８ＭＨｚ

ANSELA=0; //すべてデジタルI/O

ANSELB=0; //すべてデジタルI/O

TRISA=0; //RA全て出力(RA5は入力のみ)

TRISB=0; //RBすべて出力

PORTA=0; //RA出力すべてlow

PORTB=0; //RB出力すべてlow

}

int main() {

init();

lcd_init();//ここでDBポートのTRISが設定される

lcd_ShowString(&#8220;Hello World!&#8221;);

__delay_ms(2000);

while(1){

__delay_ms(1000);

lcd_ClearDisplay();

printf(&#8220;LCD Display Test&#8221;);//printfも使える!

lcd_MoveLine(2);

printf(&#8221; SC1602BS*B&#8221;);

__delay_ms(1500);

lcd_ClearDisplay();

printf(&#8220;A!#$&'()&#8217;@+*<>?}&#8221;);

}

return 0;

}

~~~

 [1]: http://blog.cormoran-web.com/wp-content/uploads/2014/09/lcd.helloworld.jpg
 [2]: http://blog.cormoran-web.com/wp-content/uploads/2014/09/SC1602BSBTest.png