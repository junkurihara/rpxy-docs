---
title: HTTP/3
type: docs
prev: /docs/guide/advanced/
next: /docs/guide/advanced/client_auth
weight: 1
sidebar:
  open: true
---

`rpxy`は[`quinn`](https://github.com/quinn-rs/quinn)、[`s2n-quic`](https://github.com/aws/s2n-quic)、[`hyperium/h3`](https://github.com/hyperium/h3)により、HTTP/3リクエストを処理できます。この実験的機能を有効にするには、`config.toml`に`experimental.h3`エントリを以下のように追加してください。すると、TLS有効なアプリケーションはHTTP/2とHTTP/1.1に加えてHTTP/3で提供されます（[TLSクライアント認証が有効なアプリケーション](/docs/guide/advanced/client_auth)を除く）。

```toml
[experimental.h3]
alt_svc_max_age = 3600             # 秒
max_concurrent_connections = 512   # エンドポイント/リスナーごとのH3接続数
max_concurrent_bidistream = 100    # H3接続ごとの双方向ストリーム数
max_concurrent_unistream = 100     # H3接続ごとの単方向ストリーム数
max_idle_timeout = 10              # 秒。0は無制限。
```

{{< callout type="info" >}}
`alt_svc_max_age`などのエントリの値はすべてオプションです。
{{< /callout >}}

HTTP/3の接続数上限は、HTTP/1.1とHTTP/2のみを対象とするグローバルオプション`max_clients`とは独立しています。`max_concurrent_connections`は、設定されたH3エンドポイント/リスナーごとにHTTP/3 (QUIC)接続数を個別に制限します。

HTTP/3のリクエストボディサイズ上限は、HTTP/1.1・HTTP/2と共通のトップレベルオプション`request_max_body_size`で指定します。

{{< callout type="warning" >}}
`[experimental.h3]`配下のHTTP/3専用`request_max_body_size`キーはv0.14.0で削除されました。このキーを含む設定は読み込みエラーになります。代わりにトップレベルの`request_max_body_size`を使用してください。
{{< /callout >}}
