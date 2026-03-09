---
title: インストール
type: docs
prev: /docs/guide/
next: /docs/guide/cmdopt
weight: 1
sidebar:
  open: true
---

{{< callout type="warning" >}}
**バージョン0.10.0の破壊的変更**:
- 非`watch`実行オプションが削除されました。動的な設定リロードがデフォルトで有効になりました。
- ログディレクトリの場所を指定する新しい`log-dir`オプションが追加されました。
- ログファイルがアクセスログとエラーログに分離されました。
{{< /callout >}}


## ソースからのビルド

Gitリポジトリをクローンして、実行可能バイナリを自分でビルドできます。

```bash
# Gitリポジトリのクローン
% git clone https://github.com/junkurihara/rust-rpxy
% cd rust-rpxy

# サブモジュールの更新
% git submodule update --init

# ビルド
% cargo build --release
```

ビルド後、`rust-rpxy/target/release/rpxy`に実行可能バイナリが生成されます。

{{< callout type="info" >}}
デフォルトでは`quinn`を使用してQUICとHTTP/3が有効になっています。`s2n-quic`を使用したい場合は、以下のようにビルドしてください。追加の依存関係が必要になる場合があります。

```bash
% cargo build --no-default-features --features http3-s2n --release
```
{{< /callout >}}

## Linux向けパッケージインストール (RPM/DEB)

`rpxy`のJenkins CI/CDビルドスクリプトは[./.build](https://github.com/junkurihara/rust-rpxy/tree/develop/.build)ディレクトリにあります。

Linux RPMおよびDEBのビルド済みパッケージは、[@Gamerboy59](https://github.com/Gamerboy59)が提供する[https://rpxy.gamerboy59.dev](https://rpxy.gamerboy59.dev)で入手できます。

## ビルド済みバイナリ

`amd64` (`x86_64`) と `arm64` (`aarch64`) 向けのビルド済みバイナリは[GitHub Releases](https://github.com/junkurihara/rust-rpxy/releases/latest)で入手できます。ビルド済みバイナリの命名規則は以下の通りです。

```plaintext
rpxy-${ARCH}-unknown-linux-${TARGET}-${BUILD_OPT}.tar.gz
```

| 変数 | 説明 |
| --- | --- |
| `${ARCH}` | `x86_64`または`aarch64` |
| `${TARGET}` | `gnu`または`musl` |
| `${BUILD_OPT}` | `s2n`: QUICライブラリとして`s2n-quic`を使用。 |
| | `webpki-roots`: バックエンドアプリケーションとの接続用ルート証明書バンドルとして[`WebPKI`](https://github.com/rustls/webpki)を使用。 |

{{< callout type="warning" >}}
バックエンドアプリケーションに自己署名証明書を使用している場合は、`webpki-roots`以外のバイナリを選択する必要があります。
{{< /callout >}}


{{< callout type="info" >}}
**動的設定**: バージョン0.10.0以降、`rpxy`は設定ファイルの変更を監視し、再起動なしで自動的に設定をリロードします。これがデフォルトの動作です。
{{< /callout >}}

{{< callout type="info" >}}
一部の依存関係がまだ公開されていないため、現時点では[`crates.io`](https://crates.io/)経由のインストール（`cargo install`）オプションはありません。代替として、`amd64`環境では最も簡単な方法としてDockerイメージを使用できます（[コンテナセクション](/docs/container/)を参照）。
{{< /callout >}}
