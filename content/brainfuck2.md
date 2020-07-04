---
title: Brainfuck で HelloWorld など
author: cormoran
date: 2015-01-09
categories:
  - Programing
tags:
  - language
---
せっかくインタープリターを書いたのでbrainfuckでもう少し遊びます。

<!--more-->



ただHelloWorldを表示させるのは楽しくないので好きな文字列を表示するbrainfuckソースコードを生成するプログラム（長い!）を作ってみました。

　例えば&#8221;Hello World!&#8221;を入れると以下のようなファイルが生成されます。

~~~

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.

+++++++++++++++++++++++++++++.

+++++++.

.

+++.


+++++++++++++++++++++++++++++++++++++++++++++++++++++++.

++++++++++++++++++++++++.

+++.


~~~

ループを使えばもっと短くかけるのですがそれはまた別の機会に&#8230;

以下がプログラムです。[ここ][1]にも置いています。ループ無しで（改行がなければ）一番短いプログラムを出力させているつもりです。<ins datetime="2015-01-09T08:44:15+00:00">（と思っていましたが、書いている最中に、ポインターをインクリメントしたほうが早い場合があると気付きました。メモリを一つだけ使うプログラムの中では最短だと思います。）</ins>

使い方：コンパイルして実行にテキストファイルのパス、出力したいファイル名を引数で渡す。

~~~

#include&lt;iostream&gt;
#include&lt;fstream&gt;
#include&lt;string&gt;
#include&lt;cstdlib&gt;


int main(int argc,char **argv){
  if(argc!=3){std::cerr&lt;&lt;"please input following format : inputfilepath outputfilename"&lt;&lt;std::endl;exit(0);}

  std::ifstream ifs(argv[1]);

  if(ifs.fail()){std::cerr&lt;&lt;"no such file"&lt;&lt;std::endl;exit(0);}

  std::string input="",output="";
  unsigned char value=0;
  while(getline(ifs,input)){
    input.push_back('\n');
    for(int i=0;i&lt;input.size();i++){
      int diff=abs((int)input[i]-(int)value+256) - abs((int)value+256-(int)input[i]);
      //incrementしたほうが近い場合
      if(diff&gt;0){
	while(value!=input[i]){++value;output.push_back('+');}
      }
      //decrementしたほうが近い場合
      else if(diff&lt;0){
	while(value!=input[i]){--value;output.push_back('-');}
      }
      output+=".\n";
    }
  }
  ifs.close();

  std::ofstream ofs(argv[2]);
  ofs&lt;&lt;output;
  ofs.close();

  return 0;
}
~~~

 [1]: https://github.com/cormoran/Brainfuck-interpreter