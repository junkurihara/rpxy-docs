---
title: コンテナの使用
type: docs
prev: /docs/
next: /docs/container/docker
weight: 20
---

`rpxy` の Docker コンテナイメージは、[Docker Hub](https://hub.docker.com/r/jqtype/rpxy) と [GitHub Container Registry](https://github.com/junkurihara/rust-rpxy/pkgs/container/rust-rpxy) で公開されています。

このセクションでは、コンテナ環境で `rpxy` を使うための情報をまとめています。

## 最新版とバージョン付きビルド

正式リリース時には、`main` ブランチから最新版イメージが提供されます。たとえばバージョン `x.y.z` のリリース時には、次のイメージが公開されます。

- `latest`, `x.y.z`: デフォルト機能でビルド、Ubuntu上で動作。
- `latest-slim`, `slim`, `x.y.z-slim`: デフォルト機能で`musl`ビルド、Alpine上で動作。
- `latest-s2n`, `s2n`, `x.y.z-s2n`: `http3-s2n`機能でビルド、Ubuntu上で動作。

同様に、`webpki-roots` 付きでビルドされたイメージも提供されます（例: `latest-s2n-webpki-roots` と `s2n-webpki-roots` は同じイメージを指します）。

## ナイトリービルド

ナイトリービルドは、`develop` ブランチへの push ごとに提供されます。

- `nightly`: デフォルト機能でビルド、Ubuntu上で動作。
- `nightly-slim`: デフォルト機能で`musl`ビルド、Alpine上で動作。
- `nightly-s2n`: `http3-s2n`機能でビルド、Ubuntu上で動作。

こちらも `webpki-roots` 付きイメージが用意されています（例: `nightly-s2n-webpki-roots`）。

## 注意事項

`s2n-quic` のサブパッケージが `musl` 環境でコンパイルエラーを起こすため、`nightly-s2n-slim` と `latest-s2n-slim` は現在提供されていません。
