---
title: 'ICPC2018 国内予選 参加記'
date: 2018-07-09T01:49:52+09:00
tags:
    - '参加記'
---

今年も ICPC 国内予選に出た．getting_over_32 というチームで [@takeo1116](https://twitter.com/takeo1116) と [@fwarashi](https://twitter.com/fwarashi) で参加．A,B,C,D の４完４１位（学内２位）だった．大雨の影響などで選抜ルールが変更されなければアジア地区通過らしい．

自分は A,B,D の実装をした．B を通すのが遅くなってしまったのはとても反省．

<!--more-->

去年は kimiyuki，KokiYmgch と参加してアジア地区は実質保証されていた感があったけど，今年は別チームで，それなりにがんばらないと行けないなぁという感じだった．ミスがなくて，学内の他２チームに負けなければアジア地区はなんとかなるかな，くらいの感じでいた．

## 事前戦略

事前のチーム練習は模擬地区の１回だけだったので，あまりまじめな戦略はたてられなかった．レーティング的には自分が一番高いけど，考察力的にはあまり変わらなさそうな気がしていたので，いい感じにローテーションすればいいか...と適当に思っていたが，@takeo1116 が Windows 民で Linux コマンドがだめだったので，自分と @fwarashi で実装をやることになった．

@fwarashi が環境構築，A を自分が通して，B を @fwarashi，@takeo1116 の C の考察を聞いて自分が C 通すみたいな雰囲気の戦略を立てていた気がする．が後々冷静に考えると結構厳しい気がするのでアジア地区に出られる場合は，とりあえず @takeo1116 に Linux を叩き込まないといけない．

## 本番準備

例の大雨で電車が止まっていて遠方組は大変だったが，自分は徒歩通学なので大丈夫だった．到着がだいぶ遅れてつらそうな人もいたけど，うちの大学から出た４チームは全メンバー揃って大学で出られた．

環境の準備は毎年のことだけど，PC が足りなかったり，PC のメモリが壊れていたり，OS が古かったり，プリンタが繋がらなかったりで大変だった．雨でできなかったのだけど，前日準備はとても大切...．

キーボードは @fwarashi の RealForce を使わせてもらったけど，やっぱり打ちやすかったので静電容量式のキーボードが欲しくなった．

## 本番

@fwarashi に環境構築をしてもらいつつ，モニタで自分が A を読んだ．簡単すぎて環境構築より早く解法が出たので，B も自分が読むことにした．B 読んで，とりあえずざっくり実装できそうだけどつらいなぁみたいな気分になったところで PC が空いたので A を実装（7:46 AC）．

B を @fwarashi に読んでもらっていた気がするが，実装がやるだけだけど面倒そうだったのでとりあえず自分が実装始めておくと言って勝手に始めた気がする．詳細を @fwarashi に読んで確認してもらうつもりでいたけど，この辺で情報共有がちょっと適当になった気がする．微妙に長い時間をかけてとりあえず実装できたけど領域外アクセスしてつらい気分になった．その辺で２人に投げていた C が実装できるとのことだったのでいったん PC を譲った．見返すと，紙の幅と高さ N, M を逆にしていたり，折り返しの index 計算が適当すぎて申し訳ない気分になった．その後 C もバグった（？）らしいとかで２回ほど PC を交換した．C が通ったあたりで制約の誤読がわかってまたつらくなったが，初期の想定内だったので適当に対応した．(C AC 1:03:00, B AC 1:15:06)

C が通ったあとは２人に D を投げていたので，自分は E を読むことにした．問題文は理解したが，$N <= 10^{18}$ で桁上がりが大変そう...って感じだった．考えてもしばらくかかりそうだったのと順位表を見て D は解くべきな感じだったので一度２人と問題交換した（この選択がかなり良かったはず）．D は２人は枝狩り全探索(?)みたいな感じで止まっていたので，よくわからないけどサンプルに最悪に近いケースがあって答えの組み合わせ数的にはなんとかなりそうだったので，とりあえず全探索書いた．ちょっと詰まって心配になりつつも計算が終わったので提出．(1:57:50 AC)

とりあえず４完して，上にいた TEAM_IZ も４完だったので，ワンチャン抜けるのでは？みたいな話をした．

一息ついて残りの問題内容共有した．順位表をみると E 以外はほとんど解かれていなかったのと，自分が D を実装している間に２人が E をダブリング的方向で考えていて，方向は良さそうだったので，３人で E を考えることにした（気がする？）．ただのダブリング（繰り返し２乗法）では誤差がだめだと思いつつも，とりあえず @fwarashi に実装を始めてもらって，自分と @takeo1116 でもう少し考えることにした．色々話していると，桁上がりの直前までダブリング的に進めるのを繰り返すと行けそうという気持ちになったので PC を奪って実装を始める（残り３０分＋ α くらい？）．残り１５分くらいでコア部分がだいたい実装できる．動かしてみると全然答えが違ってつらい．時間的に厳しいなぁと思いつつも色々やっていると，終了．順位表を見ていると，@takeo1116 が選抜ルールを完全に覚えていたらしく，チーム数を数えて通過したっぽいみたいな話をした．

E は見直していたら入力が仮数部のみであることに気づいたり， 1. の部分をとりあえず放置したままにしていたりしてもうちょっと時間が必要な感じだった．

終わってから見返すと，解くべきだった順番は A -> C -> (B or D) -> (E or F?) ... という感じ．WA が 0 だったのは良かった．

最近ほとんど精進できていなかったことを考えるとまあまあの出来かもしれないけど，実装力不足感が否めないので，とりあえず実装力精進の必要性を感じた．今回の B くらいは思考停止でバグなく実装したい．

## 提出したプログラムたち

### A

やるだけという感じ．何も考えていないけど，浮動小数は怖かったので整数比較した．

```c++
#include<bits/stdc++.h>

using namespace std;
using ll=long long;
int main() {
  while(true) {
    ll N; cin >> N;
    if(N == 0) break;
    vector<ll> A(N);
    ll sum = 0;
    for(int i = 0; i < N; i++) cin >> A[i];
    for(int i = 0; i < N; i++) sum += A[i];
    int ans = 0;
    for(int i = 0; i < N; i++) {
      ans += A[i] * N <= sum;
    }
    cout << ans << endl;
  }
  return 0;
}
```

### B

最初に `N` `M` を逆にしてしまって，その後も`N` `M` と `X` `Y` の対応を取るのがつらかった．`H` `W` にしてくれ．（`H` `W` で実装すれば良いだけの話...）

```c++
#include<bits/stdc++.h>
#define rep(i, N) for(int i = 0; i < (int)(N); i++)

using namespace std;
using ll=long long;
int main() {
    while(true) {
        int N, M, T, P; cin >> N >> M >> T >> P;
        // cerr << N << " " << M  << " " << T << " " << P << endl;
        if(N == 0 and M == 0) break;
        int MM = M;
        vector<int> D(T), C(T);
        rep(t, T) cin >> D[t] >> C[t];

        vector<int> X(P), Y(P);
        rep(p, P) cin >> X[p] >> Y[p];

        vector<vector<int>> G(M * 100, vector<int>(N * 100));
        rep(m, M) rep(n, N) G[m][n] = 1;
        N *= 100;
        M *= 100;
        int sx = 0, sy = 0;
        rep(t, T) {
            int d = D[t], c = C[t];
            // cerr << "!" << t << endl;
            if(d == 1) {
                rep(i, c) {
                    rep(y, M) {
                        int tmp = G[y][i + sx];
                        G[y][i + sx] = 0;
                        G[y][sx + c + c - i - 1] += tmp;
                    }
                }
                sx += c;
            } else if(d == 2) {
                rep(i, c) {
                    rep(x, N) {
                        int tmp = G[i + sy][x];
                        G[i + sy][x] = 0;
                        G[sy + c + c - i - 1][x] += tmp; // TODO
                    }
                }
                sy += c;
            } else assert(0);
            // cerr << endl << endl;
            // rep(i, MM) {
            // 	rep(j, N) cerr << G[i][j] << " ";
            // 	cerr << endl;
            // }
        }
        rep(p, P) {
            int x = X[p], y = Y[p];
            cout << G[y + sy][x + sx] << endl;
            // TODO
        }
    }
    return 0;
}
```

### D

```c++
#include<bits/stdc++.h>
#define rep(i, N) for(int i = 0; i < (int)(N); i++)

using namespace std;
using ll=long long;
using pii=pair<int,int>;
int main() {
    while(true) {
        int N; cin >> N;
        if(N == 0) break;
        int M; cin >> M;

        vector<int> X(M), Y(M);
        rep(i, M) {
            cin >> X[i] >> Y[i];
            X[i]--;
            Y[i]--;
        }

        vector<vector<int>> win(N, vector<int>(N));

        rep(i, M) {
            win[X[i]][Y[i]]++;
            win[Y[i]][X[i]]--;
        }

        // cerr << "!--" << endl;
        // rep(i, N) {
        //   rep(j, N) {
        // 	cerr << win[i][j] << " ";
        // 	assert(-1 <= win[i][j] and win[i][j] <= 1);
        //   }
        //   cerr << endl;
        // }

        vector<pii> left; // y x
        rep(i, N) rep(j, i) if(win[i][j] == 0) left.emplace_back(i, j);
        vector<int> sums(N);
        rep(i, N) rep(j, N) if(win[i][j] > 0) sums[i]++;

        function<ll(int, vector<int>&)> dfs = [&] (int i, vector<int> &sums) {
            if(i >= left.size()) {
                bool ok = true;
                rep(i, N) ok &= sums[i] == N / 2;
                return ok ? 1ll : 0ll;
            }
            ll ans = 0;
            pii p = left[i];
            if(++sums[p.first] <= N / 2) {
                ans += dfs(i + 1, sums);
            }
            sums[p.first]--;
            if(++sums[p.second] <= N / 2) {
                ans += dfs(i + 1, sums);
            }
            sums[p.second]--;
            return ans;
        };
        // cerr << "ans ";
        cout << dfs(0, sums) << endl;
    }
    return 0;
}
```
