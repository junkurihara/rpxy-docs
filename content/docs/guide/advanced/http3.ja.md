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
alt_svc_max_age = 3600
request_max_body_size = 65536
max_concurrent_connections = 10000
max_concurrent_bidistream = 100
max_concurrent_unistream = 100
max_idle_timeout = 10
```

{{< callout type="info" >}}
`alt_svc_max_age`などのエントリの値はすべてオプションです。
{{< /callout >}}
