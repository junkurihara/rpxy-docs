---
title: HTTP/3
type: docs
prev: /docs/guide/advanced/
next: /docs/guide/advanced/client_auth
weight: 1
sidebar:
  open: true
---

`rpxy` can serve HTTP/3 requests thanks to [`quinn`](https://github.com/quinn-rs/quinn), [`s2n-quic`](https://github.com/aws/s2n-quic) and [`hyperium/h3`](https://github.com/hyperium/h3). To enable this experimental feature, add an entry `experimental.h3` in your `config.toml` like follows. Then, any TLS-enabled application can be served with HTTP/3 in addition to HTTP/2 and HTTP/1.1 [except for the applications with TLS client authentication](/docs/guide/advanced/client_auth).

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
Any values in the entry like `alt_svc_max_age` are optional.
{{< /callout >}}
