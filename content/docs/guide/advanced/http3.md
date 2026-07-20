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
alt_svc_max_age = 3600             # sec
max_concurrent_connections = 512   # H3 connections per endpoint/listener
max_concurrent_bidistream = 100    # bidirectional streams per H3 connection
max_concurrent_unistream = 100     # unidirectional streams per H3 connection
max_idle_timeout = 10              # sec. 0 means an infinite timeout.
```

{{< callout type="info" >}}
Any values in the entry like `alt_svc_max_age` are optional.
{{< /callout >}}

The HTTP/3 connection limit is independent of the global `max_clients` option, which covers HTTP/1.1 and HTTP/2 only: `max_concurrent_connections` caps HTTP/3 (QUIC) connections separately for each configured H3 endpoint/listener.

The request body size limit for HTTP/3 is the top-level `request_max_body_size` option, shared with HTTP/1.1 and HTTP/2.

{{< callout type="warning" >}}
The HTTP/3-specific `request_max_body_size` key under `[experimental.h3]` was removed in v0.14.0. A configuration containing it now fails to load; use the top-level `request_max_body_size` instead.
{{< /callout >}}
