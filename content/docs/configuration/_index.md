---
title: Configuration Options
type: docs
prev: /docs/
# next: /docs/
weight: 30
# sidebar:
#   open: true
---

This page is a quick reference for the keys used in [`config-example.toml`](https://github.com/junkurihara/rust-rpxy/blob/main/config-example.toml).
For step-by-step setups, see [Basic Setups](/docs/guide/basics) and [Advanced Usage](/docs/guide/advanced).

{{< callout type="info" >}}
This page is intentionally dictionary-like. Detailed behavior and deployment examples are covered in the guide pages linked below.
{{< /callout >}}

## Quick Navigation

- [Quick Navigation](#quick-navigation)
- [Minimal Example](#minimal-example)
- [Global Settings](#global-settings)
- [Application Definition](#application-definition)
  - [`[apps.<app_name>]`](#appsapp_name)
- [TLS Options](#tls-options)
- [Reverse Proxy Rules](#reverse-proxy-rules)
- [Upstream Entries](#upstream-entries)
- [Upstream Option Values](#upstream-option-values)
- [Experimental Settings](#experimental-settings)
  - [`[experimental]`](#experimental)
  - [`[experimental.h3]`](#experimentalh3)
  - [`[experimental.cache]`](#experimentalcache)
  - [`[experimental.acme]`](#experimentalacme)
  - [`[experimental.tcp_recv_proxy_protocol]`](#experimentaltcp_recv_proxy_protocol)
- [Related Guides](#related-guides)

## Minimal Example

```toml
listen_port = 80
listen_port_tls = 443

[apps.app1]
server_name = "app1.example.com"
tls = { tls_cert_path = "./server.crt", tls_cert_key_path = "./server.key" }

[[apps.app1.reverse_proxy]]
upstream = [
  { location = "app1.local:8080" },
]
```

## Global Settings

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `listen_port` | No | None | TCP port for plaintext HTTP. At least one of `listen_port` or `listen_port_tls` must be specified. |
| `listen_port_tls` | No | None | TCP port for HTTPS/TLS. Required if you serve TLS, use ACME, or enable HTTP/3. |
| `https_redirection_port` | No | Same as `listen_port_tls` when TLS is enabled | External HTTPS port used in `301` redirects and `Alt-Svc` headers when the public HTTPS port differs from `listen_port_tls`, for example behind container port mapping or firewall redirection. |
| `tcp_listen_backlog` | No | Internal default | TCP listen backlog for HTTP/1.1 and HTTP/2 listeners. |
| `max_concurrent_streams` | No | Internal default | HTTP/2 concurrent stream limit per connection. |
| `max_clients` | No | Internal default | Total concurrent client limit across HTTP/1.1, HTTP/2, and HTTP/3. |
| `listen_address_v4` | No | `0.0.0.0` | IPv4 address to bind listeners to. |
| `listen_address_v6` | No | None | IPv6 address to bind listeners to, for example `[::]`. If omitted and `listen_ipv6 = true`, binds to `[::]`. If omitted and `listen_ipv6` is `false` or unset, IPv6 is disabled. |
| `listen_ipv6` | No | `false` | If `true`, bind to `[::]` when `listen_address_v6` is not specified. |
| `default_app` | No | None | Fallback app name for unmatched plaintext HTTP requests. This applies only to HTTP. Unknown HTTPS requests are still rejected because no TLS server name can be selected. |

## Application Definition

All backend applications are defined under `[apps]`.

### `[apps.<app_name>]`

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `server_name` | Yes | None | Hostname served by this app, for example `app.example.com`. |
| `reverse_proxy` | Yes | None | List of routing rules for this app. |
| `tls` | No | None | TLS settings for this app. If omitted, the app serves plaintext HTTP only. |

## TLS Options

These options live under `apps.<app_name>.tls`.

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `tls_cert_path` | Yes for static TLS | None | PEM certificate path for this app when using static certificates. |
| `tls_cert_key_path` | Yes for static TLS | None | PEM private key path for this app. The key must be in PKCS8 format. |
| `https_redirection` | No | `true` when both `listen_port` and `listen_port_tls` are set | Per-app HTTP-to-HTTPS redirect switch. If you serve HTTPS only, do not set this option. |
| `client_ca_cert_path` | No | None | CA certificate path for mTLS client authentication. See [Client Authentication](/docs/guide/advanced/client_auth). |
| `acme` | No | `false` | If `true`, certificates are issued and renewed automatically through ACME instead of `tls_cert_path` and `tls_cert_key_path`. See [ACME (Let's Encrypt) Integration](/docs/guide/advanced/acme). |

## Reverse Proxy Rules

Each app can contain one or more `[[apps.<app_name>.reverse_proxy]]` entries.

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `path` | No | None | Path prefix to match, such as `"/api"` or `"/static"`. Longest-prefix match wins. |
| `replace_path` | No | Preserve original path | Rewritten prefix forwarded upstream. |
| `upstream` | Yes | None | List of backend destinations. |
| `load_balance` | No | `none` | Backend selection strategy. Supported values are `none`, `round_robin`, `random`, and `sticky`. |
| `upstream_options` | No | None | List of request-forwarding behaviors. See [Upstream Options](/docs/guide/advanced/upstream_options). |

## Upstream Entries

Each item in `upstream = [...]` accepts:

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `location` | Yes | None | Backend host and port, for example `"backend.internal:8080"` or `"www.example.com"`. |
| `tls` | No | `false` | If `true`, the upstream connection uses HTTPS. If omitted or `false`, HTTP is used. |

## Upstream Option Values

See [Upstream Options](/docs/guide/advanced/upstream_options) for detailed explanations of each option. The table below summarizes the supported values for `upstream_options`.

| Value | Effect |
| --- | --- |
| `keep_original_host` | Preserve the incoming `Host` header. This is the default behavior. |
| `set_upstream_host` | Replace the `Host` header with the upstream host. |
| `upgrade_insecure_requests` | Add `Upgrade-Insecure-Requests: 1` if it is not already present. |
| `force_http11_upstream` | Force HTTP/1.1 for upstream connections. |
| `force_http2_upstream` | Force HTTP/2 for upstream connections. |
| `forwarded_header` | Generate the RFC 7239 `Forwarded` header in addition to the default `X-Forwarded-*` headers. |

## Experimental Settings

The `[experimental]` table controls optional and advanced features.

### `[experimental]`

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `ignore_sni_consistency` | No | `false` | If `true`, relax SNI consistency checks for TLS backends. Keeping it `false` is recommended. |
| `connection_handling_timeout` | No | `0` | Total connection handling timeout in seconds. `0` means no timeout. |

### `[experimental.h3]`

Add this table to enable HTTP/3 support. See [HTTP/3](/docs/guide/advanced/http3).

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `alt_svc_max_age` | No | Internal default | `Alt-Svc` max-age in seconds. |
| `request_max_body_size` | No | Internal default | Maximum HTTP/3 request body size in bytes. |
| `max_concurrent_connections` | No | Internal default | Concurrent QUIC connection limit. |
| `max_concurrent_bidistream` | No | Internal default | Bidirectional QUIC stream limit. |
| `max_concurrent_unistream` | No | Internal default | Unidirectional QUIC stream limit. |
| `max_idle_timeout` | No | Internal default | QUIC idle timeout in seconds. `0` means infinite timeout. |

### `[experimental.cache]`

Add this table to enable the hybrid response cache. See [Caching](/docs/guide/advanced/cache).

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `cache_dir` | No | `./cache` | Cache directory path, relative to the current working directory. |
| `max_cache_entry` | No | `1000` | Maximum number of cache entries. |
| `max_cache_each_size` | No | `65535` | Maximum size in bytes for a single cacheable response. |
| `max_cache_each_size_on_memory` | No | `4096` | Maximum size in bytes for keeping a cached response in memory. Larger cached responses are stored as files. Set `0` to always use file cache. |

### `[experimental.acme]`

Add this table when any app uses `tls = { acme = true }`. See [ACME (Let's Encrypt) Integration](/docs/guide/advanced/acme).

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `email` | Yes | None | Contact email for ACME account registration. |
| `dir_url` | No | Let's Encrypt production directory | ACME directory URL. |
| `registry_path` | No | `./acme_registry` | Storage directory for retrieved certificates and account data. |

### `[experimental.tcp_recv_proxy_protocol]`

Add this table to accept inbound HAProxy PROXY protocol headers from a trusted L4 proxy. See [PROXY Protocol](/docs/guide/advanced/proxy_protocol).

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `trusted_proxies` | Yes | None | Non-empty list of trusted proxy CIDR ranges, for example `["127.0.0.1/32", "::1/128"]`. |
| `timeout` | No | `50` ms | Timeout in milliseconds for receiving the PROXY header. Setting `0` falls back to an internal `5s` timeout. |

## Related Guides

- [Command Line Options](/docs/guide/cmdopt)
- [Basic Setups](/docs/guide/basics)
- [Upstream Options](/docs/guide/advanced/upstream_options)
- [Client Authentication](/docs/guide/advanced/client_auth)
- [HTTP/3](/docs/guide/advanced/http3)
- [Caching](/docs/guide/advanced/cache)
- [ACME (Let's Encrypt) Integration](/docs/guide/advanced/acme)
- [PROXY Protocol](/docs/guide/advanced/proxy_protocol)
