---
title: Upstream Options
type: docs
prev: /docs/guide/advanced/cache
next: /docs/guide/advanced/post_quantum
weight: 45
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

By default, `rpxy` automatically adds the following forwarding headers to every request without any configuration:

| Header | Description |
| --- | --- |
| `X-Forwarded-For` | Appended with the client's IP address. If already present, the new IP is comma-appended. |
| `X-Forwarded-Proto` | Set to `https` or `http` based on the incoming connection. Added only if not already present. |
| `X-Forwarded-Port` | Set to the listening port. Added only if not already present. |
| `X-Forwarded-SSL` | Set to `on` or `off` based on the incoming connection. |
| `X-Real-IP` | Set to the client's IP address. |
| `X-Original-URI` | Set to the original request URI. |

The `forwarded_header` option adds the RFC 7239 `Forwarded` header **in addition to** these default headers:

```plaintext
Forwarded: for=192.0.2.1;proto=https;host=app1.example.com
```

{{< callout type="info" >}}
If the incoming request already contains a `Forwarded` header, `rpxy` will update it for consistency even without the `forwarded_header` option. The option controls whether to **generate a new** `Forwarded` header when one does not already exist.
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
