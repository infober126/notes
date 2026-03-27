---
layout: post
title: "EC2踏み台＋ポートフォワードでAWS REST APIをcurlしたら、Hostヘッダーで詰まった話"
date: 2026-03-27
category: tech
tags: [AWS, EC2, ECS, ALB, curl, SSM, ポートフォワード]
published: true
---

# EC2踏み台＋ポートフォワードでAWS REST APIをcurlしたら、Hostヘッダーで詰まった話

AWS の REST API にローカルから curl でリクエスト投げたくて、
ポートフォワードで繋いだら ALB のルーティングでハマった。
同じとこで詰まる人がいるかもなので書いておく。

---

## やりたかったこと

本番環境の ECS 上で動いてる REST API に、ローカルマシンから動作確認で直接リクエストを投げたかった。
ECS は直接インターネットに公開されてないので、EC2 を踏み台にして SSM Session Manager でポートフォワードしてアクセスする構成。

流れはこんな感じ：

```
ローカル → SSM Session Manager（EC2踏み台）→ ALB → ECS（REST API）
```

---

## SSO でログインしてセッション開始

踏み台への接続は SSH キーじゃなくて AWS SSO + SSM Session Manager を使ってる。

まず SSO でログイン：

```bash
aws sso login --profile my-profile
```

ブラウザが開いてログインが完了したら、SSM でポートフォワードのセッションを張る：

```bash
aws ssm start-session \
  --target i-xxxxxxxxxxxxxxxxx \
  --document-name AWS-StartPortForwardingSessionToRemoteHost \
  --parameters '{"host":["<ALBのDNS名>"],"portNumber":["80"],"localPortNumber":["8080"]}' \
  --profile my-profile
```

これで `localhost:8080` に投げたリクエストが、EC2 経由で ALB に転送されるようになる。

---

## いざ curl！…なんか変なレスポンスが返ってくる

セッションが繋がったので、curl でリクエストを投げてみた。

```bash
curl http://localhost:8080/api/v1/health
```

なんか期待してたレスポンスじゃないものが返ってくる。
エラーというわけじゃないけど、明らかに違うコンテンツ。

「え、ポートフォワードの設定間違えた？」とか思いながら、
`--verbose` オプションつけて色々確認してみた。

```bash
curl -v http://localhost:8080/api/v1/health
```

リクエストヘッダーを見てみると...

```
> Host: localhost:8080
```

あ、`Host` が `localhost:8080` になってる。

---

## 原因：ALB がホスト名でルーティングしてた

ALB（Application Load Balancer）のルーティングルールを調べてみたら、こうなってた：

- ホスト名に `test` が含まれる → テスト用 ECS へ
- 本番用のホスト名（`api.example.com` など）→ 本番 ECS へ

ポートフォワード経由で叩くと、`Host` ヘッダーが `localhost:8080` になる。
ALB はそのホスト名でルールを判定するので、本番用のルールにマッチしない。
結果、デフォルトルールのテスト用 ECS に飛ばされてた、というオチ。

なるほどな〜ってなった。

---

## 解決：`-H` で Host ヘッダーを直接指定する

`curl` の `-H` オプションでヘッダーを上書きするだけでよかった。

```bash
curl -H "Host: api.example.com" http://localhost:8080/api/v1/health
```

これで ALB が正しいルールにマッチして、本番の ECS にリクエストが届いた。
ちゃんとしたレスポンスが返ってきた時はテンション上がった。

---

## 各機能の簡単な説明

### AWS SSO（IAM Identity Center）

AWS のシングルサインオン機能。
`aws sso login --profile <プロファイル名>` でブラウザ認証できて、複数アカウントの切り替えも楽。

> 🔗 [https://docs.aws.amazon.com/ja_jp/singlesignon/latest/userguide/](https://docs.aws.amazon.com/ja_jp/singlesignon/latest/userguide/)

---

### AWS SSM Session Manager

SSH キーや踏み台サーバーへの直接接続なしで、EC2 にセキュアにアクセスできるサービス。
`aws ssm start-session` でポートフォワードのセッションも張れる。
ポート開放が不要なのでセキュリティ的にも◎。

> 🔗 [https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager.html](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager.html)

---

### EC2（Amazon Elastic Compute Cloud）

AWS の仮想サーバー。
今回は SSM 経由でポートフォワードの中継点（踏み台）として使用。

> 🔗 [https://docs.aws.amazon.com/ja_jp/ec2/](https://docs.aws.amazon.com/ja_jp/ec2/)

---

### ALB（Application Load Balancer）

HTTP/HTTPS をレイヤー7（アプリケーション層）で振り分けるロードバランサー。
**ホストベースルーティング**でホスト名ごとに転送先を変えられる。
今回のハマりポイントはここだった。

> 🔗 [https://docs.aws.amazon.com/ja_jp/elasticloadbalancing/latest/application/](https://docs.aws.amazon.com/ja_jp/elasticloadbalancing/latest/application/)

---

### ECS（Amazon Elastic Container Service）

Docker コンテナを AWS 上で動かすサービス。
今回は本番用とテスト用でクラスターが分かれていて、ALB のルールで振り分けられてた。

> 🔗 [https://docs.aws.amazon.com/ja_jp/ecs/](https://docs.aws.amazon.com/ja_jp/ecs/)

---

### curl の `-H` オプション

HTTP リクエストに任意のヘッダーを追加・上書きできるオプション。

```bash
curl -H "ヘッダー名: 値" <URL>
```

`Host` ヘッダーは HTTP/1.1 の必須ヘッダーで、リクエストの宛先ホストを示す。
ALB みたいなホストベースルーティングをするサーバーは、このヘッダーを見てルーティングを決める。

> 🔗 curl 公式: [https://curl.se/docs/manpage.html](https://curl.se/docs/manpage.html)  
> 🔗 MDN - Host ヘッダー: [https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Host](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Host)

---

## まとめ

| 問題 | 原因 | 解決 |
|------|------|------|
| ポートフォワード経由のcurlが意図しないECSに届く | `Host` ヘッダーが `localhost:8080` になり、ALBのルールにマッチしなかった | `curl -H "Host: api.example.com"` でヘッダーを上書き |

ポートフォワードで繋いだ時点で「つながった！」って満足しちゃって、
HTTP ヘッダーまで気が回ってなかったのが敗因。
ALB のルーティングはホスト名を見てるってことをちゃんと意識しないとダメだなと学んだ。