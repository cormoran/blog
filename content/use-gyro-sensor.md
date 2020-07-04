---
title: ジャイロセンサを使ってみた
author: cormoran

date: 2014-09-08

categories:
  - 電子工作
---

秋月電子の[小型圧電振動ジャイロモジュール][1]を使ってみました。今回はただ値を読み取ってLCDに表示しているだけです。LCDの記事は[こちら][2]。

<!--more-->

## ジャイロセンサ

今回使ったジャイロセンサはアナログ出力なので前回の加速度センサのような面倒な処理は不要で、マイコンのAD変換で電圧を読み取るだけです。

## サンプルプログラム

MPLABX,XC8使用、PIC16F1827を使っています。lcdのモジュールは[ここ][2]にあるものと同じファイルへのリンクです。（実はLCDのライブラリ完成前にやったので動作は未確認）

AD変換については基準電圧を内蔵固定電圧の4倍 4.096Vにしています。ジャイロモジュールの静止時電圧が1.35V、PICの分解能が10bit(1024段階)なので静止時はたぶん 1.35/4.096*1024 =337.5 周辺の値が表示されると思うのですがあまり良くわかっていません。プログラムも今回は適当ですので気が向いたら手直しします。

[lcd library (github)](https://github.com/cormoran/AVR)

~~~
#include"lcd.h"
#include<stdio.h>

#define AN9set 0b00100101
#define AN10set 0b00101001

#pragma config CLKOUTEN=OFF , FOSC=INTOSC , FCMEN=OFF , IESO=OFF , BOREN=ON , PWRTE=ON , WDTE=OFF , MCLRE=OFF , CP=OFF , CPD=OFF
#pragma config PLLEN=OFF , STVREN=ON , WRT=OFF , BORV=HI , LVP=OFF

void init(void){
    OSCCON = 0b01110010; // 内部クロック８ＭＨｚ
    ANSELA = 0;          // AN0-AN4全てデジタルI/O
    ANSELB = 0b00001100; // AN9,AN10をアナログ入力に
    TRISA  = 0;          // RA全て出力(RA5は入力のみ)
    TRISB  = 0b00001100; // RBはアナログ入力のところを入力に
    PORTA  = 0;
    PORTB  = 0;

    ADCON1 = 0b10010011 ;    // 読取値は右寄せ、A/D変換クロックはFOSC/8、fvrをリファレンスに
    ADCON0 = AN9set ;    // アナログ変換情報設定(AN9から読込む)

    FVRCON=0b10000011;//AD変換用fvrを4.096Vで有効化
    __delay_us(5) ;

}

int main() {
    int addata[2];
    init();
    lcd_init();

    printf("GyroscopeTest");
    lcd_MoveLine(2);
    printf(" Sensor Test");
    __delay_ms(3000);
    lcd_ClearDisplay();
    lcd_CursorAtHome();

    while(1){
        ADCON0 = AN9set ;
        __delay_us(10);
        GO_nDONE = 1 ;	      // アナログ値読取り開始
        while(GO_nDONE) ;        // 読取り完了まで待つ
        addata[0] = ADRESH ;
        addata[0] = ( addata[0] &lt;&lt; 8 ) | ADRESL ;

        ADCON0 = AN10set ;
        __delay_us(10);
        GO_nDONE = 1 ;	      // アナログ値読取り開始
        while(GO_nDONE) ;        // 読取り完了まで待つ
        addata[1] = ADRESH ;
        addata[1] = ( addata[1] &lt;&lt; 8 ) | ADRESL ;

        lcd_CursorAtHome();
        printf("1:%4d 2:%4d",addata[0],addata[1]);//LCDに表示
        __delay_ms(500);
    }
    return 0;
}
~~~

## 動作の様子

<iframe width="640" height="360" src="https://www.youtube.com/embed/bEdToYpOSVg?feature=oembed" frameborder="0" allowfullscreen=""></iframe>

 [1]: http://akizukidenshi.com/catalog/g/gK-04912/
 [2]: /blog/use-char-lcd
