---
title: Upstream Options
type: docs
prev: /docs/guide/advanced/cache
next: /docs/guide/advanced/trusted_proxies
weight: 5
sidebar:
  open: true
---

`rpxy` provides several options to control how request messages are forwarded to upstream backend applications. These are specified per reverse proxy entry using the `upstream_options` array.

## Configuration

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = 'backend.local:8080' },
]
upstream_options = [
  "set_upstream_host",
  "forwarded_header",
]
```

## Available Options

### `keep_original_host`

Preserve the original `Host` header from the client request. This is the **default behavior** and takes priority over `set_upstream_host` if both are specified.

### `set_upstream_host`

Overwrite the `Host` header with the upstream hostname (e.g., `backend.local:8080`). Ignored if `keep_original_host` is also specified.

### `upgrade_insecure_requests`

Add the `Upgrade-Insecure-Requests: 1` header to the forwarded request if not already present.

### `force_http11_upstream`

Force HTTP/1.1 for the upstream connection. Mutually exclusive with `force_http2_upstream`.

### `force_http2_upstream`

Force HTTP/2 for the upstream connection. Mutually exclusive with `force_http11_upstream`.

### `forwarded_header`

Generate and add the standard [`Forwarded` header (RFC 7239)](https://www.rfc-editor.org/rfc/rfc7239) to the forwarded request.

By default, `rpxy` automatically rebuilds the following forwarding headers on every request without any configuration:

| Header | Description |
| --- | --- |
| `X-Forwarded-For` | Rebuilt from the forwarding chain. By default (no `trusted_forwarded_proxies` configured), incoming values are ignored and the header carries only the immediate peer's IP address. When the immediate peer is a trusted proxy, the incoming chain is preserved and normalized. |
| `X-Forwarded-Proto` | Overwritten with `https` or `http` based on the incoming connection. |
| `X-Forwarded-Port` | Overwritten with the listening port. |
| `X-Forwarded-Host` | Overwritten with the original client-visible host. A client-supplied value is never forwarded as-is; treat it as observational data only. |
| `X-Forwarded-SSL` | Set to `on` or `off` based on the incoming connection. |
| `X-Real-IP` | Set to the client's IP address. |
| `X-Original-URI` | Set to the original request URI. |

{{< callout type="warning" >}}
Since v0.12.0, no proxy is trusted by default: forwarding headers received from the preceding peer are discarded and rebuilt from the immediate peer address. To preserve the chain built by a trusted load balancer or CDN in front of `rpxy`, configure the global `trusted_forwarded_proxies` option. See [Trusted Forwarded Proxies](/docs/guide/advanced/trusted_proxies).
{{< /callout >}}

The `forwarded_header` option adds the RFC 7239 `Forwarded` header **in addition to** these default headers:

```plaintext
Forwarded: for=192.0.2.1;proto=https;host=app1.example.com
```

{{< callout type="info" >}}
If the incoming request already contains a `Forwarded` header, `rpxy` will update it for consistency (according to `trusted_forwarded_proxies`) even without the `forwarded_header` option. The option controls whether to **generate a new** `Forwarded` header when one does not already exist.
{{< /callout >}}

## Example

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = 'www.example.com', tls = true },
  { location = 'www.example.org', tls = true },
]
load_balance = "round_robin"
upstream_options = [
  "set_upstream_host",
  "forwarded_header",
  "force_http11_upstream",
]

[[apps.app1.reverse_proxy]]
path = '/api'
replace_path = '/'
upstream = [
  { location = 'api.internal:3000' },
]
upstream_options = [
  "upgrade_insecure_requests",
]
```
