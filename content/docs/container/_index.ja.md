---
title: コンテナの使用
type: docs
prev: /docs/
next: /docs/container/docker
weight: 20
---

`rpxy`のDockerコンテナイメージは[Docker Hub](https://hub.docker.com/r/jqtype/rpxy)と[GitHub Container Registry](https://github.com/junkurihara/rust-rpxy/pkgs/container/rust-rpxy)の両方でホストされています。

このセクションでは、コンテナ環境での`rpxy`の使用方法について説明します。

## 最新版とバージョン付きビルド

最新ビルドは新しいバージョンがリリースされた際に`main`ブランチから提供されます。例えば、バージョン`x.y.z`がリリースされると、以下のイメージが提供されます。

- `latest`, `x.y.z`: デフォルト機能でビルド、Ubuntu上で動作。
- `latest-slim`, `slim`, `x.y.z-slim`: デフォルト機能で`musl`ビルド、Alpine上で動作。
- `latest-s2n`, `s2n`, `x.y.z-s2n`: `http3-s2n`機能でビルド、Ubuntu上で動作。

上記と同様に、`webpki-roots`付きでビルドされたイメージも提供されます（例: `latest-s2n-webpki-roots`と`s2n-webpki-roots`が同じイメージにタグ付け）。

## ナイトリービルド

ナイトリービルドは`develop`ブランチへのプッシュごとに提供されます。

- `nightly`: デフォルト機能でビルド、Ubuntu上で動作。
- `nightly-slim`: デフォルト機能で`musl`ビルド、Alpine上で動作。
- `nightly-s2n`: `http3-s2n`機能でビルド、Ubuntu上で動作。

上記と同様に、`webpki-roots`付きでビルドされたイメージも提供されます（例: `nightly-s2n-webpki-roots`）。

## 注意事項

`s2n-quic`のサブパッケージに`musl`でのコンパイルエラーがあるため、`nightly-s2n-slim`や`latest-s2n-slim`はまだ提供されていません。
