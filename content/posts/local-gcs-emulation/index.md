---
title: "ローカル環境でGCSをエミュレートしよう"
date: 2020-10-04T20:54:17+09:00
draft: false
summary: fake-gcs-serverを使いローカル環境でGCSのエミュレートをします。
---

## はじめに

はじめまして。[sueken5](https://twitter.com/Sueken51)です。

この記事ではGCPサービスの一つであるGCSをローカルでエミュレートしgolangからそれを利用する方法を紹介します。

## fake-gcs-server

[fake-gcs-server](https://github.com/fsouza/fake-gcs-server)

gcsのエミュレーターはgcp公式が提供しているものがないので[fake-gcs-server](https://github.com/fsouza/fake-gcs-server)を利用します。

dockerイメージが提供されているので次のコマンドで実際に動かすことができます。

`docker run -d --name fake-gcs-server -p 4443:4443 fsouza/fake-gcs-server`

ローカルで利用するにあたり重要なフラグを紹介します。

`--scheme`はhttpsかhttpかを決めることできるフラグです。ローカルで使う分にはhttpでいいので`--schema http`と指定します。

次に`--public-host`です。このフラグはGCSのオブジェクトのPublicURLの働きをするものになっています。今回は`localhost`で参照したいので`--public-host localhost`と指定します。

次にバケットの設定です。

`/data`にオブジェクトが保存されるようになっており、バケット名をディレクトリ名にし`/data/bucket-name`となるようにボリュームをマウントします。これをしないと保存はできるのですが、参照処理ができません。注意してください。

## 実際に動かす

実際に動かしてみます。今回はgolangを利用します。実際のコードは[こちら](https://github.com/sueken5/local-gcs-emulator-example/blob/main/main.go)を参考にしてください。

実際に動かす前に`STORAGE_EMULATOR_HOST`という環境変数を設定します。
storage.NewClientは初期化時に`STORAGE_EMULATOR_HOST`という環境変数が存在していたらその値をホストに使うようにします。

> [golang gcs storage 初期化処理](https://github.com/googleapis/google-cloud-go/blob/c66c5b8799db244755c353f47d159830c8ee4e7c/storage/storage.go#L105-L113)

実際に動かして見ると `http://localhost:4443/bucket-name/onjname` でアクセスできることが確認できます。

## まとめ

gcsのローカルエミュレートを紹介しました。簡単に利用できるのでテストやローカル環境の構築に使っていきたいです。