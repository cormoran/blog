+++
date = "2016-05-02T23:50:10+09:00"
title = "Windowsでストレスのない開発環境を構築するメモ（の一部）"

categories = ["Programing"]
+++

ロボットの部にあるWindowsが使いにくくてつらかったので（自分が）使いやすいようにいろいろやってみました。

あまり大したことはやってないのですが、自分向けに、部の人向けにメモ程度に。やっとことをざっとまとめるだけで、深いことは書きません。

<!--more-->

普段はMacユーザーなのでWindowsで普通はこうする！とかいうのあまり知りません。もっといい方法等があるかもしれません。

## やったこと

1. chocolatey （パッケージマネージャー）を入れる
2. git を使えるようにする
3. ネットワーク設定を切り替えるバッチファイルを作る
4. いろいろ...


## 1.[chocolatey](https://chocolatey.org)を入れる

Ubuntuだったら apt-get とか aptitude , Mac だったらbrew, mac ports などがあります。これを使うとコマンド一つでソフトのインストールができ、個別のソフトをいちいちインストーラを落としてきて実行して...っていうことをする必要がなくなります。ライブラリの依存関係を勝手に解決してくれたり、アップデートやアンインストールの管理もしてくれます。

インストール方法は[公式サイト](https://chocolatey.org)で。

コマンドプロンプトからインストールする場合は C:\> ... っていう方をコピーして貼り付け、powershellからインストールする場合は PS:\>の方をコピーして貼り付けるとあとは勝手にやってくれます。**コマンドプロンプト、powershellは管理者権限で立ち上げておく必要があります。**

日本語でわかりやすそうなサイト : [Chocolateyを使った環境構築の時のメモ](http://qiita.com/konta220/items/95b40b4647a737cb51aa)

これで

~~~
choco install <パッケージ名>
cinst <パッケージ名>
~~~

などのコマンドを打つことでソフトをインストールできます。

とりあえずGUI版のchocolateyを入れておきます。CUIからやればいいのですが、windowsのコマンドプロンプトあまり好きではない（使い慣れてないだけ）のでGUI使いたくなる時がたまにあります。あと、chocolateyはXMLでパッケージリストを書くと、他の環境でそれを読み込んで一気に全部インストールができるという機能があるのですが、そのファイルを出力できるので便利です。（cuiからでもできるの？）


~~~
cinst chocolateygui
~~~

chocolateyでインストールできるパッケージ一覧は[ここ](https://chocolatey.org/packages)にありますが、chocolatey guiでも似たような感じで検索できます。

今回インストールしたパッケージ一覧

~~~
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="chocolatey" version="0.9.9.12" />
  <package id="ChocolateyGUI" version="0.13.2" />
  <package id="git" version="2.8.1" />
  <package id="git.install" version="2.8.1" />
  <package id="sourcetree" version="1.8.3" />
  <package id="PowerShell" version="5.0.10586.20151218" />
  <package id="python2-x86_32" version="2.7.11" />
  <package id="python3-x86_32" version="3.5.1" />
</packages>
~~~

メジャーなプログラムはすでにPCには入っているものが多かったので欲しかったものと、chocolateyで入れたかったものだけ入れました。
opencvとかライブラリ系のパッケージもあって、opencv3を入れてみたのですが、異常に時間がかかったり、VS2015-64bit用しか入らなかったり（これは妥当かもしれない）、chocolateyに登録されなかったりと、よくわからなかったので諦めました。

## 2. gitを入れる

といっても、上ですでに入れています。

~~~
cinst git
~~~

これでgitコマンドが使える上に、git bashというシェルが手に入ります。キーバインドがunixスタイルで、よく使うunixコマンドが使えるようになります。

本気でunix風に使うならcygwinとかを入れればいいのですが、まあ、これでもちょっとしたことには十分です。

あと、GUIからもできるようにsource treeを入れておきます。最初はcuiでいいじゃないかと思っていたのですが、使ってみると以外といいです。

## 3.ネットワーク設定を切り替えるバッチファイルを作る

これは完全にお家事情なのですが、大会でのLAN環境はサブネットが各チームに割り当てられていて、貸与された機器で実験をするときにはいちいちIPの設定を変えないといけません。やるだけなのですが、何度もやることになると結構面倒。調べるとPowerShellが結構強くてこのあたりの設定ができるようなのでやってみました。サーバーとかで使うのでしょうか。

参考になるサイト :

[PowerShellの使い方(OS設定編)](http://qiita.com/Kirito1617/items/aed439bcb66c63489337)

[PowerShellによるIPアドレスの設定及び変更](http://qiita.com/hanakara_milk/items/1197ac9b91fa1fa3abab)

以下はIPアドレスを192.168.0.1に固定するバッチファイルの例

~~~
@powershell -NoProfile -ExecutionPolicy Unrestricted "$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt 1})-join\"`n\");&$s" %*&goto:eof

New-NetIPAddress -InterfaceIndex 4 -IPAddress "192.168.0.1" -AddressFamily IPv4 -PrefixLength 24
~~~

InterfaceIndex はwifiを固定するか、有線lanを固定するか等設定するアダプタによって異なります。 Get-NetAdapte コマンドで調べられます。

また、冒頭の @powershell ... はコマンドプロンプトからpowershellを呼び出すおまじないで、調べたら出てきて、うまく動くのでよくわからず使っています。これを使うことで、ダブルクリックで実行できる.batファイルからpowershellコマンドを呼び出すことができます。powershellスクリプトはダブルクリックとかでは無理（？）な様子。ただしバッチファイルは**管理者権限で実行しないといけません**。

以下はDHCPを有効にする例。

~~~
@powershell -NoProfile -ExecutionPolicy Unrestricted "$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt 1})-join\"`n\");&$s" %*&goto:eof

Set-NetIPInterface -InterfaceIndex 4 -Dhcp Enabled
Disable-NetAdapter -Name "Wi-Fi"
Enable-NetAdapter -Name "Wi-Fi"
~~~

なんかすぐには固定していたIPを明け渡さない（？）みたいなのでアダプタを一旦無効化して、再度有効化することで強制的に新しいIPをもらいに行かせています。

