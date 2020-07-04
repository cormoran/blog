---
title: AVR UART通信 on mega168p
author: cormoran
date: 2015-02-25
categories:
  - 電子工作

---
ATmega168pでUART通信をしてみました。

<!--more-->



通信したのはBluetoothモジュールである RN-42 。このモジュールはUSARTとUSBインターフェースを持っており、それらの通信を無線化することができます。なのでAVR側で必要なのは普通のUSART(UART)通信だけあり、マイコンで非常に簡単にBluetooth通信をすることができます。

　Bluetooth通信関連の記事は後日書くことにして、今回はUART通信に絞って書きます。

## UART? USART?

googleで&#8221;avr USART&#8221;と検索すると&#8221;もしかして avr UART&#8221;と表示され、なんか嫌な気分になるのですが、atmega168pに搭載されているモジュールはUSARTです。USARTとUARTの違いは前者がクロック同期式の通信をサポートしているのに対し、後者は非同期のみに対応していることです（たぶん）。ちなみに

USART : Universal Synchronous Asynchronous Receiver Transmitter

UART : Universal Asynchronous Receiver Transmitter

で英語を見れば大体わかります。今回やることはモジュールはUSARTなのでUSARTと言うべきだろうと思いますがライブラリを全部UARTと書いてしまっており、通信も非同期式でやっているのでUARTっていう言葉でいきます。（適当）

## ライブラリ

今回作ったライブラリは

<a href="http://blog.cormoran-web.com/doxygenfiles/2015/avr_uart_lib" title="AVR UART Library" target="_blank">ライブラリ説明用</a>

<a href="https://github.com/cormoran/AVR.git" title="AVR_github" target="_blank">ソースコード</a>

ソースコードは上のリンクの Library -> UART -> firmware にあります。Doxygenを初めて使ってみたので説明は適当ですが。また、まだいろいろ試している途中なので構成を変える可能性があり、リンクエラーになるかもしれません。

上のライブラリはAVRのUSARTモジュールの設定をまとめていることに加え、送受信バッファを独自に作って文字列の送信をうまくできるようにしています。

ATmega168p以外のavrマイコンでもヘッダファイルのレジスタ関連のマクロ部分を変えれば使えると思います。他のマイコンで使ったら順次設定を追加していくつもりです。

備忘録を兼ねて簡単に説明を。以下は部分抜粋、貼り合わせです。コメントを追加して説明としています。

~~~

//割り込みを利用して自動で送受信を行う。

//mega168pは受信データを1byteしかハード的には保持しないので

//自分でバッファを用意してそのバッファを介してデータをやり取りする。

//バッファは使う場所をくるくる回して使う。

//（ローリングバッファっていうらしい。）

//バッファサイズ

#define RCVBUF_SIZE 25

#define SNDBUF_SIZE 25

volatile signed char rcvbuf[RCVBUF_SIZE]={};

volatile signed char sndbuf[SNDBUF_SIZE]={};

// _w:次に書き込むバッファの位置

// _r:次に読み取る場所

volatile unsigned char sndbuf\_cnt\_w=0,sndbuf\_cnt\_r=0;

volatile unsigned char rcvbuf\_cnt\_w=0,rcvbuf\_cnt\_r=0;

volatile char sndbuf\_over\_flg=0;//バッファ溢れを防ぐためにつかう

void UART_init(){

//とりあえずレジスタを0にしてから設定しているが冗長的かもしれない

UCSRB = 0 ;

UCSRC = 0 ;

//受信完了、送信準備完了割り込み許可

UCSRB |= bSET(RXCIE) | bSET(UDRIE);

// 0b(0)11 DataBit長 8bit

UCSRC |= bSET(UCSZ1) | bSET(UCSZ0);

//（ちなみに）非同期通信、パリティ検査禁止

//ボーレート設定

UBRR = F_CPU/16/BAUD &#8211; 1 ;

//RX,TXピンを有効化、送受信許可

UCSRB |= bSET(RXEN) | bSET(TXEN);

}

///受信完了時割り込み

//受信データを受信バッファに入れる。

//現状バッファが溢れると過去の1周分のデータが消える。

ISR(USART\_RX\_vect){

rcvbuf[ (int)rcvbuf\_cnt\_w++ ] = UDR ;

if( rcvbuf\_cnt\_w >= RCVBUF\_SIZE ) rcvbuf\_cnt_w = 0 ;

}

///送信準備完了時割り込み

//送信はputchar関数（２種類ある）でバッファ溢れ対策をしているので

//不意にデータが消える心配はない。overflgを追加してるのでちょっと複雑。

//ここのコードだけではわからないと思う。

ISR(USART\_UDRE\_vect){

if( sndbuf\_over\_flg || sndbuf\_cnt\_r != sndbuf\_cnt\_w ){

UDR = sndbuf[ (int)sndbuf\_cnt\_r++ ];

if( sndbuf\_cnt\_r >= SNDBUF\_SIZE ) sndbuf\_cnt_r = 0;

sndbuf\_over\_flg = 0;

}

else UCSRB &= bCLR(UDRIE);

}

~~~

main.cは簡単な使い方のサンプルになっているので詳しくはご覧ください。