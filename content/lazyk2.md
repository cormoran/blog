---
title: Lazy K　〜インタラクティブシェルの導入〜　 (Mac)
author: cormoran
date: 2015-02-26
categories:
  - Programing
tags:
  - language

---

前回記事[Lazy K に挑む　〜インタープリタの導入〜　 (Mac)][1]の続きで今回は Lazy K インタラクティブシェルの導入について書きます。今回の分は参考にできる記事がなかったので自分でいろいろやってみました。

<!--more-->

## インタラクティブシェルのインストール（前話）

適当に検索していると

[Lazy K][2]

といういい感じのサイトにたどり着きました。２枚目のスライドに &#8220;combinator-interactive&#8221; をインストールせよとあります。もうちょっと見ていくとこいつがインタラクティブシェルだとわかります。（なんかファイルからプログラムを読み込めそうな感じのことが書いていたのですが、よくわからなかったのとlazy.cppがあるのであまり調べていません）

~~~

cabal update

cabal install combinator-interactive

~~~

でインストールできるとあるのでやってみます。（ちなみに cabal というのは Haskell のパッケージ管理システムみたいなもので HomeBrew を入れていれば ghc と cabal-install をインストールするとつかえます。 ）すると、途中までは進むのですが lens というパッケージが入れられないとかいうエラーでインストールできません。少し手間取ったのですが力技で解決しました。

　問題は

~~~

cabal info combinator-interactive

~~~

とやってみると依存関係の項目で lens >=3.8 && <=4 とあるのですが、 lens 最新バージョンは4.7で、何かの問題で3.8〜4の間のやつが入れられない、という感じでした。(lensのinfoでは3.9とか3.10が使えるよと書いてあったのですが...)この辺は自分にはよくわからなかったので lens4.7 でも動くだろうという楽観的推測の元、ソースを落としてコンパイルしてみました。多分うまくいきました。

## インタラクティブシェルのインストール（本番）

さっきの cabal info で Homepage の項目にgithubのURLが載っていたので行ってみるとソースがありました。

[github.com/fumieval/combinator-interactive][3]

~~~

git clone https://github.com/fumieval/combinator-interactive

~~~

として、ソースを一式落としてきてそのフォルダ内で

~~~

ghc -o lazy main.hs

~~~

としてコンパイルしてみます。するとここは自分が Haskeller かどうかによるのですが○○がimportできないよと言われると思います。さっきの cabal info を見て Dependencies にある base,lens,template-haskell,th-lift, void, trifecta, bytestring,directory, cereal, mtl, containers,combinator-interactive の内 combinator-interactive 以外のインストールしていないやつをcabal install で入れていきます。（インストールされているかは cabal list &#8211;installed で確認できる。）

　そしてコンパイルすると最後にData.Combinatorがないよと言われます。こいつは落としてきたファイルに含まれているやつなのでソースを書き換えます。Haskellほとんど使っていないのでよくわからないのですが以下のようにしたらコンパイルできました。

~~~

import &#8220;combinator-interactive&#8221; Data.Combinator

#上の部分の &#8221; &#8221; 部を消して下のようにする

import Data.Combinator

#SyntaxHighlighter が Haskell対応してない!!!

~~~

実行してみると

~~~

./lazy

lazy> Ix

x

lazy> Kx y

x

lazy> Sx y z

x z (y z)

~~~

動作しました。最初のさわりはインタラクティブシェルを使う方がやりやすそうな気がします。

　今後はちょっと本気で Lazy K を触るためにコンビネータ理論やラムダ式について少し知識をつけていく予定です。

 [1]: http://blog.cormoran-web.com/2015/02/27/lazy-k-1-interpreter/ "Lazy K に挑む　〜インタープリタの導入〜　 (Mac)"
 [2]: http://shared.botis.org/yurufuwa-lazy/out.html#/
 [3]: https://github.com/fumieval/combinator-interactive
