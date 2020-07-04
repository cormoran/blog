---
title: "IQ1でもできる！機械学習研究の実験管理！"
date: 2019-12-15T01:31:54+09:00
---

[IQ1 Advent Calendar 2019](https://adventar.org/calendars/4115) の 15 日目の記事です。

この記事では、機械学習研究における実験の管理方法について、

- IQ1 の人間が経験から編み出した方法
- ネットで調べた様々な Best Practice

についてお話します。

<!--more-->

## IQ1 の方法

IQ1 でない人々は、諸々が最強っぽくて、バージョン管理がなくても、いろいろ適当にやっていても生きていられるようですが、IQ1 は IQ が 1 なのでいろいろ工夫していかないとすぐに死んでしまいます。

以下、私がやっているソースコード・実験結果の管理方法について、IQ1 工夫ポイントを交えつつ書きます。

### 前提：計算機の環境

計算資源に、統一的に管理されていないサーバーを使うか、クラウドを使うか、スパコンみたいにジョブ管理システムがあるものを使うか等によって話が変わってくる部分があると思います。

私の環境は以下のような感じです。

- 物理数十台の計算機
- 計算資源管理は適当
  - 基本は共有で、空いているサーバー、GPU を阿吽の呼吸で使う
- ファイルシステムは計算機ごと
  - nfs とかない

### ソースコード管理

バージョン管理を最大限に活用して、注意ポイントと記憶すべきことを減らします

- 開発プログラムは手元の PC に置く
- DropBox リアルタイム同期（研究室 PC、自宅 PC、ラップトップ）
  - DropBox にはバージョン履歴機能があって、git commit 前のデータを復元できます
  - 他のクラウドドライブはクライアントソフトがクソだったり、Linux 未対応だったりするので DropBox 一択です
- 手動で git バージョン管理
  - IQ1 にはある程度まとまった git log が必要なので、commit は切りの良いタイミングでします
- 保存と同時に、rsync で計算機サーバーに単方向同期（デバッグ実行用）
  - デバッグに任意のサーバーが使えて、どのサーバーに何があるか意識する必要もなくなります
  - 自動で同期されないと、IQ1 は同期を忘れたまま実行して死にます
- 実験は、専用ディレクトリに git clone して動かす
  - 実験ログと git commit が対応して、実験時の状態を忘れても良くなります。
  - 自動同期によって、意図しない動作が起こるのを防げます。
  - CI や、簡単なプログラムで自動化することもできます（[例](https://blog.cormoran-web.com/blog/2019/12/gpu-job-runner/#3-%E3%82%B7%E3%82%A7%E3%83%AB%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%83%88%E3%81%A7-git-base-%E3%81%AA%E5%AE%9F%E8%A1%8C%E7%AE%A1%E7%90%86)

### ディレクトリ構造・実装

趣味です

- 実装と実験スクリプトを分ける
  - 実装は、できるだけ相互依存を排除して、再利用しやすい独立したモジュールにする
    - 実験が進むと、「僕が考えた最強の機械学習ライブラリ」になる
    - 実装はできるだけ DRY を守る
  - 実験スクリプトは引数の受け取りと、実装を組み合わせるだけの薄いものにする
- 実験スクリプトは２種類作る
  - ハイパーパラメータ等が引数で渡せるメインプログラム（[cifar10 識別器学習の例](https://gist.github.com/cormoran/d65b92981131b1ad7dab612e362c1005)
  - それに渡す引数がハードコードされた実行コマンドのスクリプト
    - IQ1 は記憶力がないので、実際に実行するコマンドがソースコードに残っていないといろいろ忘れます
- 結果集計・レポート出力スクリプトを作る
  - 結果を Excel などに１つ１つコピペしていると、IQ1 は必ずミスします
  - jinja2 など、テンプレートエンジンを使って結果のサマリーをテキストで出力し、git で管理しています
- 実行コマンドのスクリプトとその結果の集計ファイルは同じ場所に置く
  - results とか別ディレクトリを掘ると行き来がしづらい

ソースコードは、最終的に java でよく見る xml みたいな、DI になるんだと思います（なった）。

私の実装は、外から使うクラスはすべて json で扱える型のみを引数とするコンストラクタを実装していて、一つの json ファイルからモデルや学習のクラスを生成できるようになっています。
また、それらの引数には type hint を必ずつけ、クラス生成前に[typeguard](https://pypi.org/project/typeguard/)等で型チェックがされるようにしています。IQ1 に型チェックは必須！

```sh
# とあるプロジェクトのディレクトリ構造
.
├── datasets
│   ├── imagenet
│   └── cifar100
├── experiments: 実験スクリプト
│   ├── <exp name1>
│   │  ├── <exp script1>.py : main program
│   │  ├── 01_<exp script1>.sh : run main program with specific argsscript1
│   │  ├── 01_<exp script1>_summarize.py : collect results and output 01_<exp script1>.md
│   │  └── 01_<exp script1>.md : summarized result of exp
│   └── <exp name 2>
├── modules
│   ├── classifiers
│   ├── criteria
│   ├── dataloaders
│   ├── datasets
│   ├── evaluators
│   ├── lib
│   ├── manager
│   ├── models
│   ├── networks
│   ├── trainers
│   └── ...
├── scripts
│   └── サーバーに同期するスクリプトとか
├── survey
├── notes
├── tests
├── Readme.md
├── requirements.txt
├── setup.cfg: yapf setting
└── venv: python virtual environment
```

### 実行結果・ログファイル

- ログは実行ごとに `<実験ID>_<試行回数>_<実験を表す human readable な文字列>` というディレクトリを作る
  - 実験 ID は実行コマンドのスクリプトに１体１対応するランダムな文字列です
  - 実験 ID がない場合、`実験を表す human readable な文字列`が他の連続部分文字列になるとき、grep が面倒になります
    - 例）`kmeans` という実験のあとに `kmeans_with_XXX` とかするのはよくあると思います
    - これを回避するのは、IQ1 には多分無理です
  - 昔は実行時間や、実行ごとのランダム ID をつけていましたが、同一実験の判定にパースが必要で面倒なのでやめました
- 実行結果・ログは sshfs でマウントしたディレクトリに書き込む
  - 全サーバーで、一つのサーバーのログディレクトリを sshfs でマウントしています
  - 以前はローカル保存 + rsync 収集でした
    - （IQ1 には？） `rsync --delete` でうまくファイルをフィルタするのが無理だけど、ディレクトリを消したいことがよくある
    - 複数サーバーでいろいろプログラムが動くので、ログファイルを簡易ロック機構に使いたい
  - ネットワークが死ぬと死にます...
- バックアップをちゃんと取る
  - IQ1 なので、先日、間違って**数ヶ月分のログを全部消し**ましたが、バックアップをちゃんと取っていたので生き延びました
  - 普段は `rm` 封印して `trash` するようにしているのですが、IQ1 は自信過剰なので `/bin/rm -rf` とか `rsync --delete`をラフに使ってしまう

ログの内容ですが、実行環境や git commit hash, git diff 等基本的なものに加えて、実装で触れた、主要クラスに渡す引数も json 化して保存しています。

ログファイル名の命名に失敗して、実験ごとの違いがよくわからなくなったときは、この json ファイルの diff を取ることで、大まかなの設定の違いがわかってうれしいです。

### 実験管理

IQ1 は記憶力がないので、実験を動かし始めたことをよく忘れます。

特に、複数のアイデアやパラメータを変える実験を並列で動かすと、必ずいくつかは忘れます。

- 実験を始めたら、Todoist に必ずタスクを登録する
- 上で紹介した実行コマンドのスクリプト内でタスク追加が実行されるように自動化がよいです
- あまり細かいレベルで ToDo を自動追加すると、 ToDo が増えすぎて IQ1 の認識力を超えるので注意
  - やる前から容易に想像できるのですが、IQ1 なので経験済みです
  - API rate limit に弾かれる問題もあります

### テスト

- 自身を持って言えるほどは書いていない、CI も回してない
  - バグらせそうな部分や、初期のやる気に満ち溢れていた部分は書かれている
- IQ1 的には結構まずいね...
- 締め切り前後にバグを見つけて死ぬ未来が見える

## ネットで調べた様々な Best Practice

ここまでは、私の方法を紹介しましたが、少し賢者になって、他の人の方法も調べたのでまとめておきます。

実質おすすめブックマーク集です。

### 基本的な指針

[Best Practices for Scientific Computing](https://arxiv.org/abs/1210.0530) は必読な気がします。

特別なことは書いていませんが、大事なポイントがしっかり抑えられていて良いです。リーダブルコードを読んでいる気分になります。

要点まとめは、[こちらのブログ](https://takuti.me/note/data-science-project-structure/#%E3%81%BF%E3%82%93%E3%81%AA%E3%81%AF%E3%81%A9%E3%81%86%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E3%81%AE%E3%81%8B) が良い感じです。日本語が好きな人は、まずこれを読むのがおすすめです。

<small>[Good Enough Practices in Scientific Computing](https://arxiv.org/abs/1609.00037) という論文もあって、こちらはもう少し具体的な内容に踏み込んだものですが、あまり心には刺さりませんでした。</small>

### 研究のワークフロー

深層学習関連では、Dustin Tran さん （元 Blei 研、現 Google Brain、Edward 作者）が研究ワークフロー、ディレクトリ管理について[ブログ](http://dustintran.com/blog/a-research-to-engineering-workflow)を書いています。

上を参考にした日本語記事だと [研究ワークフロー](https://enomotokenji.github.io/2018/04/15/research_workflow.html)があります。

guicho271828 さんが書いている以下の記事は、IQ1 的に割と共感できました。

- [計算機科学研究での記録のとり方](https://qiita.com/guicho271828/items/cf393ad5180cf059ba5b)
- [情報系研究者のための研究ノート](https://qiita.com/guicho271828/items/9307ae12248329b71f12)

> "Your computer should always be running experiments, since there's no reason that you should be working harder than your computer...

### フォルダ構成

プロジェクトのフォルダ構成に関してはいろいろな人が自分の方法をさらしていますが、とりあえず [Cookiecutter Data Science](https://drivendata.github.io/cookiecutter-data-science/) 一読しておくと良さそうです。

機械学習の実装により踏み込んだものとしては以下は目を通すと良さそうです。

- [機械学習で泣かないためのコード設計 2018](https://www.slideshare.net/takahirokubo7792/2018-97367311)
- [データ分析をするときのフォルダ構成をどうするのか問題について](https://www.st-hakky-blog.com/entry/2017/03/24/140738)

公開されている論文の実装は...というと、依存ライブラリのバージョンがわからなかったり、if 文の嵐だったり、複雑な抽象クラスを継承（まあわかるけど）していたりして、厳しい気持ちになることが多いです。

この実装よさげ、とかいうものがあればおしえていただけると嬉しいです。

自分が今まで見たものだと、[sngan_projection](https://github.com/pfnet-research/sngan_projection) は configs が良さげに感じました。

### その他の参考文献

- [研究者流コーディングの極意](http://www.chokkan.org/publication/coding-for-researchers.pdf)
- [Patterns for Research in Machine Learning](http://arkitus.com/patterns-for-research-in-machine-learning/)
- [kaggle discussion](https://www.kaggle.com/general/4815)
- [これでもう悩まない！機械学習のためのフォルダ構成テンプレートを使おう。](https://qiita.com/qmiyajun/items/5039b97a159b9f521e49)
- [私が機械学習研究をするときのコード・データ管理方法](https://qiita.com/ysekky/items/3db54349452dd8a336fb)
- [俺の機械学習プロジェクト管理をさらす](https://qiita.com/wataoka/items/496aebe51ff59f4a9685)

## 最後に

> 愚か者は経験に学び、賢者は歴史に学ぶと言う...君が愚か者でないことを祈ろう

IQ1 の人へ

> 愚か者は愚か者らしく、何もかも経験で学んでみるがいい..それが理解への早道だ

はやく賢者になりたい...

---

`sed '/s/QI1/cormoran/g'`
