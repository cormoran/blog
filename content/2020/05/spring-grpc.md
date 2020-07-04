---
title: "grpc-spring-boot-starter 関連のソースコードを読んだ"
date: 2019-12-08T18:02:23+09:00
draft: true
---

spring, spring boot, grpc 初心者が spring で grpc を使ったときのソースコードリーディングメモ

## Spring Boot で grpc

- github で調べると、割といろいろな人が grpc の spring boot starter を作っている
  - <https://github.com/search?q=grpc-spring-boot-starter>
- なんかスターが 1.4k なものが 2 つあって不思議
  - <https://github.com/yidongnan/grpc-spring-boot-starter>
    - initial commit 30 Dec 2016
    - commit 700, fork 400, star 1.4k
  - <https://github.com/LogNet/grpc-spring-boot-starter>
    - initial commit 28 Jan 2016
    - commit 300, fork 300, star 1.4k
    - LogNet は会社っぽい
  - 2020/05 時点で両方更新されている

### LogNet さんの starter

yidongnan さんのやつに比べて機能がシンプル

- grpc サーバーの設定と、サービスの自動注入
- interceptor 対応

Spring の細かい挙動を十分把握してないけど、[GRpcServerRunner](https://github.com/LogNet/grpc-spring-boot-starter/blob/master/grpc-spring-boot-starter/src/main/java/org/lognet/springboot/grpc/GRpcServerRunner.java) が grpc サーバーの管理をやっている

grpc サーバー起動時の処理は

1. @GRpcGlobalInterceptor のついたクラスを集めて
2. HealthService 追加して
3. @GRpcService のついたクラスを集めて追加して
4. 設定されてれば reflection 追加して
5. サーバーを起動

見た感じ、grpc の通信があったときの処理は spring とは完全に独立している

`io.grpc.ServerBuilder` がサーバーを作るので、リクエストに対する挙動はすべてこいつが担いそう、ここから先は `grpc-java` の世界

## yidongnan 　さんの starter

機能が多い

- grpc クライアントの自動注入
- scoped bean
- sercurity
