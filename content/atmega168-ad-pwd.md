---
title: ATmega168pを使う ~PWMとAD変換~
author: cormoran

date: 2015-02-21
categories:
  - 電子工作
---
先日[春休みの心得][1]で&#8221;ダミヤンを作る&#8221;という目標を書きましたが、ATmega168pはその心臓部に使う予定です。しかし、実は自分は今までPICばかり使っていてAVRは半年ほど前に１度使っただけです。そこで今回はAVRの使い方を思い出すこととダミヤンを作るための準備を兼ねてPWM出力とA/D変換を使ってみました。

<!--more-->



<small><br /> ダミヤン：レスキューロボットコンテストで使われる災害での要救助者を模した人形。レスコンのサイトは<a href="http://www.rescue-robot-contest.org/" title="レスコン" target="_blank">こちら</a>。</small>

## ATmega168p 概要

・8bit 28ピンマイコン

・最高20MHz

・16KBのフラッシュメモリ

・1KBのSRAM

・512BのEEPROM

・DIO、10bit A/D変換、USART、SPI、TWI(I2C)、QTouchなど&#8230;

　秋月によるとATmega168の低消費電力版らしい。Arduinoによく使われるのはmega168、mega328pらしいがこいつらは基本的な機能もピン配置も同じなのでちょっと頑張れば168pでArduino互換機が作れるらしい。（先日ちょっと挑戦したがMacの何かのせいで簡単にはいかず断念中。また記事書くかもしれません。）

　実は328pはフラッシュメモリが168pの２倍ほどあって価格も秋月で50円差なのでちょっと買うなら328pがいいのですが、某ロボ研になぜか168pが大量にあったので仕方なくこいつを使っています。（ライトに使うなら168pで多分十分です）

## LEDを光らせる

　PWMの前に少しだけ簡単なやつを。0.5sごとにLEDを点滅させます。速いのが好きなのでヒューズビットを以下のコードにあるように設定して8MHzでavrを動作させています。PB0にLEDをつないでおけば点滅するはずです。（実はここは動作確認してません..）

~~~
//Fuse : High 0xDF
//       Low  0xE2
//8MHz 内部クロック、1/8分周期は使わない

#define F_CPU 8000000L    //8MHz これはdelayを使うのに必要。util/delay.hをインクルードする前に定義しておく
#include &lt;avr/io.h&gt;
#include &lt;util/delay.h&gt;

int main(void)
{
    DDRB  = 0xff ; //ポートBをすべて出力
    PORTB = 0 ;    //ポートBをすべてLowに
    while(1){
        PORTB ^= (1&lt;&lt;PB0);  //PB0の出力を反転
        _delay_ms(500);     //500ms (=0.5s) 待つ
    }
    return 0;
}
~~~

## PWMとA/D変換

AD変換の値によって&#8221;周波数&#8221;を変えています。普段は周波数固定でデューティ比を変えることが多いと思います。そっちのサンプルはそのうち..

~~~
#include &lt;avr/io.h&gt;
#include &lt;util/delay.h&gt;

#define F_CPU 8000000L    //8MHz

#define bSET(x) (1&lt;&lt;(x))     // bit x のみが1
#define bCLR(x) (~(1&lt;&lt;(x)))  // bit x のみが0
#define is_SET(x,y) ((x) & (1&lt;&lt;(y)))

/*
 * TMR1の設定(PWM)
 * 出力PWMは  周波数 clk / 8 * OCR1A
 *           Duty  High:Low = OCR1B : OCR1A
 *
 */
void TMR1_init(){

    TCNT1=0;
    TCCR1A = 0;
    TCCR1B = 0;

    TCCR1A |= bSET(WGM11) | bSET(WGM10)  ; // 0b(11)11 OCR1AをTOPをする高速PWM
    TCCR1B |= bSET(WGM13) | bSET(WGM12)  ; // 上の括弧部分

    TCCR1A |= bSET(COM1B1) ; // 0b10    OC1B出力は比較一致でLow,BottomでHigh
                             //         (ちなみに)OC1Aは無効 0b00


    ICR1 =0xffff;

    OCR1A=0x00ff;  //タイマのトップ値の設定

    OCR1B=0x00ff;     //比較一致の値

    //割り込みは不可（多分デフォルト？）
    TCCR1B |= bSET(CS11) ;   // 0b010   Timer1のクロックはclkを8分周 start
}


/*
 * A/D変換の設定
 * ADC1を入力にしてみる
 */
void AD_init(){

    ADMUX  = 0 ;
    ADMUX |= bSET(REFS0) ;    // 0b01 基準電圧AVCC
    ADMUX |= bSET(MUX0) ;     // 0b0001 A/D変換機にADC1ポートを接続

    ADCSRA  = 0 ;
    ADCSRA |= bSET(ADEN);    // A/D変換使用許可
                              // (ちなみに)入力クロックはCK/2

    DIDR0  = bSET(ADC1D);     // ADC1のポートのデジタル入力禁止（省電力のため推奨）


}

void init(){
    TMR1_init();
    AD_init();
    DDRB = 0xff;//全てのポートを出力に
    DDRC = 0xff ;//& bCLR(PC1); //PC1はA/D用入力
    PORTB=0xff;
}

int main(void)
{
    init();

    while(1){
        int adin;

        ADCSRA |= bSET(ADSC) ;        //A/D変換開始
        while(is_SET(ADCSRA,ADIF)==0) ;  //変換終了まで待機
        adin  = ADC ;
        adin=adin&lt;&lt;6;

        OCR1A = adin ;

        adin/=2;
        OCR1B = adin ;
    }

    return 0;   /* never reached */
}
~~~

 [1]: http://blog.cormoran-web.com/2015/02/17/promise-of-spring-vacation/ "春休みの生活心得"
