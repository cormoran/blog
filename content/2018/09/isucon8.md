---
title: 'ISUCON8 予選'
date: 2018-09-16T23:40:29+09:00
tags:
    - '参加記'
---

ISUCON8 の予選に参加して敗退した話.

<!--more-->

自分と, [@fwarashi](https://twitter.com/fwarashi), [@takeo1116](https://twitter.com/takeo1116) の３人で参加した．Max が 12xxx 点で，最終は 8xxx 点だった．予選敗退．

@takeo1116 さんに linux 慣れてもらうという名目で ICPC のメンバーで参加したけど，途中から完全に放置してしまったので申し訳ない...

## 準備

去年ゆるふわ参加して予選敗退していたので，今年は割と本気で本選通過を目指していた．

メンバーの他の二人はゆるふわ参加っぽいので，ひとりで ISUCON7 の問題を使って netdata 見たり，nginx, mysql のログを解析を確認したりしていた．

ansible を初めて触って，楽しくなって色々書いたけど，環境が変わったり焦ったりすると使えなくなりそうなので捨てるなどした．

ベンチマーク前にログローテートしていい感じにローカルに集めるスクリプトなどをいくつか用意はした．

## 本番

サーバーに入ったら CentOS & H2O でつらい気持ちになった．（Ubuntu 用でしかセットアップを試していなかった & H2O でログ解析ツール使えるかわからない）

色々バックアップを取ったり，ソースコードを手元に落として git 管理など各種準備をしながら，ログ解析とかロードバランサ的に nginx に変えるのが良さそうとかいう話をした．

言語は go か python という話をしていたけど，周りが python の方が使い慣れているのと，wsgi_lineprof を使いたかったので最初は python で動かしてログをとった．

その後，思考停止で nginx に切り替えたら csv 返す API とかで fail していったん nginx を諦めた．H2O のログを alp or kataribe で解析するのはできなかった（alp ならもう少し調べたらできたっぽい？）ので，この時の ningx のログを頼りに雰囲気を掴んだ．

プログラム眺めているとループとかいっぱいしていて python だと辛そうだったのと，ログ解析から DB 周りの修正が重要そうだったので最初は python でやりつつ， go にも同じ改良を入れて動かしてみるか...みたいな感じになった．（pprof は事前に試してなかったのと，wsgi_lineprof の方が便利っぽい先入観があったので使わなかった)

line profile の結果を見て，とりあえず getEvent の N+1 を改善したり，reservation 用に軽量版 getEvent を実装した．

python に実装してよくなったことを確認して， go で殴ったら 8000~11000 点くらいが出るようになった．（14 時〜15 時くらい？）

そこそこ順位が高かったのでちょっとうれしくなった．

このあたりで，とりあえずサーバー３台使えるようにしておきたいという話になって，アプリは任せて，自分は残り２台のサーバーの準備を始めた．

別サーバーで nginx 構成をもう一回試してなんか動いたので２台で nginx, app x2, db x1 を作ったが fail する．

１台で nginx に戻しても fail するので nginx で動いたというのが実は嘘だったかもしれない...

横で db の設定などを @fwarashi が触ってくれいていたけど，ベンチガチャのせいでよくわからない感じになっていた．（DB のデッドロックが原因だったらしい？）

３台で自在に構成を変えられるようにはなって，nginx はもうだめっぽいことがわかったので，17 時くらいからとりあえず１台でちゃんと動くことを確認しようということになって動かすが，たまに fail して？？となった．

その後，色々構成を変えて動かしてみたがよくわからず，時間切れ．最後は h2o+app と db の２台構成で動かしていた．

最後の方のベンチガチャで 12xxx を引いたが，最終は 8xxx 点だった．

nginx わからん．

## 反省

振り返ってみると，サーバーは初期構成のまま１台だけ使って，アプリ・DB の改善に注力するのがよかった気がする．問題的にも，チームメンバーの実力的にも．

去年も思った気がするけど，学生枠のボーダーはかなり低くて，アプリ，DB の重い部分をあと１，２個改善(or ワンチャンでベンチガチャ)できていれば通過だったっぽいので．（それができないのだけど...）

３台構成にしても，現状の負荷的にアプリ側を改善しないとだめというのはずっと感じていて，口ではずっと言っていただけに悔しい．

個人としては，インフラ周りに時間を消費してアプリ側の改善に中途半端にしか参加できなかったので，消化不良感がある．

インフラ周りの構築にそれなりに慣れているのが自分１人だったので，３台構成にする担当は自分になるのは必然だったけど，ISUCON 的なアプリ・DB の改善に一番慣れているのも自分だった気がするので，自分の使いどころを間違った感じがある．チーム戦は人数が多くて無敵感を感じるせいで逆に難しい．勝ちたかったり，メンバーを置物にしないためには，やっぱりチームで事前に練習しておいて，互いの実力を把握したり，全員が最低限のことをできるようにしておくのが大事だと思った．

あと，インフラははまりどころが多くて楽しいけど，やることは自明なので，コンテストで担当するのは実はつらいだけなのでは？ということに気づいてしまった？

後半がとてもつらかったけど，コンテストは楽しいので，またリベンジしたい．

## 改善したところ

実質これだけ

```diff
commit 77257e82b8b3a9575a9e44b64d19816020a3e27a
Author: cormoran <cormoran707@gmail.com>
Date:   Sun Sep 16 14:53:11 2018 +0900

    update go

diff --git a/torb/webapp/go/src/torb/app.go b/torb/webapp/go/src/torb/app.go
index 94d7acf..fb7d773 100644
--- a/torb/webapp/go/src/torb/app.go
+++ b/torb/webapp/go/src/torb/app.go
@@ -221,6 +221,14 @@ func getEvents(all bool) ([]*Event, error) {
 	return events, nil
 }

+func getEvent2(eventID, loginUserID int64) (*Event, error) {
+	var event Event
+	if err := db.QueryRow("SELECT * FROM events WHERE id = ?", eventID).Scan(&event.ID, &event.Title, &event.PublicFg, &event.ClosedFg, &event.Price); err != nil {
+		return nil, err
+	}
+	return &event, nil
+}
+
 func getEvent(eventID, loginUserID int64) (*Event, error) {
 	var event Event
 	if err := db.QueryRow("SELECT * FROM events WHERE id = ?", eventID).Scan(&event.ID, &event.Title, &event.PublicFg, &event.ClosedFg, &event.Price); err != nil {
@@ -239,6 +247,17 @@ func getEvent(eventID, loginUserID int64) (*Event, error) {
 	}
 	defer rows.Close()

+	rows2, err := db.Query("SELECT * FROM reservations WHERE event_id = ? AND canceled_at IS NULL GROUP BY event_id, sheet_id HAVING reserved_at = MIN(reserved_at)", event.ID)
+	defer rows2.Close()
+	reservations := map[int64]Reservation{}
+	for rows2.Next() {
+		var reservation Reservation
+		if err := rows2.Scan(&reservation.ID, &reservation.EventID, &reservation.SheetID, &reservation.UserID, &reservation.ReservedAt, &reservation.CanceledAt); err != nil {
+			return nil, err
+		}
+		reservations[reservation.SheetID] = reservation
+	}
+
 	for rows.Next() {
 		var sheet Sheet
 		if err := rows.Scan(&sheet.ID, &sheet.Rank, &sheet.Num, &sheet.Price); err != nil {
@@ -248,17 +267,14 @@ func getEvent(eventID, loginUserID int64) (*Event, error) {
 		event.Total++
 		event.Sheets[sheet.Rank].Total++

-		var reservation Reservation
-		err := db.QueryRow("SELECT * FROM reservations WHERE event_id = ? AND sheet_id = ? AND canceled_at IS NULL GROUP BY event_id, sheet_id HAVING reserved_at = MIN(reserved_at)", event.ID, sheet.ID).Scan(&reservation.ID, &reservation.EventID, &reservation.SheetID, &reservation.UserID, &reservation.ReservedAt, &reservation.CanceledAt)
-		if err == nil {
+		reservation, ok := reservations[sheet.ID]
+		if ok {
 			sheet.Mine = reservation.UserID == loginUserID
 			sheet.Reserved = true
 			sheet.ReservedAtUnix = reservation.ReservedAt.Unix()
-		} else if err == sql.ErrNoRows {
+		} else {
 			event.Remains++
 			event.Sheets[sheet.Rank].Remains++
-		} else {
-			return nil, err
 		}

 		event.Sheets[sheet.Rank].Detail = append(event.Sheets[sheet.Rank].Detail, &sheet)
@@ -571,7 +587,7 @@ func main() {
 			return err
 		}

-		event, err := getEvent(eventID, user.ID)
+		event, err := getEvent2(eventID, user.ID)
 		if err != nil {
 			if err == sql.ErrNoRows {
 				return resError(c, "invalid_event", 404)
@@ -639,7 +655,7 @@ func main() {
 			return err
 		}

-		event, err := getEvent(eventID, user.ID)
+		event, err := getEvent2(eventID, user.ID)
 		if err != nil {
 			if err == sql.ErrNoRows {
 				return resError(c, "invalid_event", 404)
```
