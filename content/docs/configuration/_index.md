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
- [Health Check Options](#health-check-options)
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
| `public_https_port` | No | Same as `listen_port_tls` when TLS is enabled | Client-visible HTTPS/HTTP-3 port used in `301` redirects and `Alt-Svc` headers when the public HTTPS port differs from `listen_port_tls`, for example behind container port mapping or firewall redirection. Renamed from `https_redirection_port` in v0.13.0. |
| `tcp_listen_backlog` | No | `1024` | TCP listen backlog for HTTP/1.1 and HTTP/2 listeners. |
| `max_concurrent_streams` | No | `64` | HTTP/2 concurrent stream limit per connection. |
| `max_clients` | No | `512` | Shared cap for accepted HTTP/1.1 and HTTP/2 TCP connections. A slot is reserved right after TCP accept and held through PROXY protocol parsing, the TLS handshake, and the whole connection lifetime. `0` rejects every HTTP/1.1 and HTTP/2 connection. HTTP/3 is not counted here; it has its own per-endpoint limit under [`[experimental.h3]`](#experimentalh3). |
| `request_max_body_size` | No | `268435456` (256 MiB) | Maximum inbound request body size shared by HTTP/1.1, HTTP/2, and HTTP/3. Accepts an integer in bytes or a string with a binary suffix such as `"256k"`, `"10m"`, or `"1g"`. Requests whose `Content-Length` exceeds the limit are rejected with `413` before any upstream contact; oversize chunked/streamed bodies are detected mid-flight. Set `0` or `"unlimited"` to disable the limit. |
| `trusted_forwarded_proxies` | No | None (no proxy trusted) | List of proxies whose incoming `X-Forwarded-*` / `Forwarded` headers are trusted. Accepts CIDR blocks and the built-in aliases `"cloudflare"`, `"fastly"`, and `"cloudfront"`. If omitted or empty, forwarding headers from preceding peers are ignored and rebuilt from the immediate peer address. See [Trusted Forwarded Proxies](/docs/guide/advanced/trusted_proxies). |
| `sticky_cookie_secret` | Required when `load_balance = "sticky"` is used | None | 32-byte secret encoded as unpadded base64url, used to seal sticky-session cookies as opaque AEAD blobs. Rotating the secret resets sticky-session affinity. A generation command is shown below the table. |
| `redact_query_in_access_log` | No | `false` | If `true`, query-string values in the access log are masked as `<redacted>` while parameter keys and the path are kept, so URLs carrying tokens or PII are not logged verbatim. |
| `listen_address_v4` | No | `0.0.0.0` | IPv4 address(es) to bind listeners to. Accepts a single string or an array of strings for binding to multiple interfaces, for example `['192.168.1.1', '10.0.0.1']`. When multiple addresses are specified, the wildcard `0.0.0.0` must not be included. Duplicate addresses are silently ignored. |
| `listen_address_v6` | No | None | IPv6 address(es) to bind listeners to. Accepts a single string or an array of strings, for example `'[::]'` or `['::1', 'fe80::1']`. If omitted and `listen_ipv6 = true`, binds to `[::]`. If omitted and `listen_ipv6` is `false` or unset, IPv6 is disabled. When multiple addresses are specified, the wildcard `::` must not be included. Duplicate addresses are silently ignored. |
| `listen_ipv6` | No | `false` | If `true`, bind to `[::]` when `listen_address_v6` is not specified. |
| `default_app` | No | None | Fallback app name for unmatched plaintext HTTP requests. This applies only to HTTP; HTTPS requests with an unknown server name are rejected unconditionally. On this fallback path, the outgoing `Host` header is force-overwritten with the default app's `server_name` (winning over `keep_original_host` / `set_upstream_host`), and the untrusted original host is exposed only in `X-Forwarded-Host` / `Forwarded: host=`. Backends must treat those values as untrusted observational data. |

A `sticky_cookie_secret` value can be generated as follows:

```bash
openssl rand -base64 32 | tr '+/' '-_' | tr -d '=\n'
```

## Application Definition

All backend applications are defined under `[apps]`.

### `[apps.<app_name>]`

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `server_name` | Yes | None | Hostname served by this app, for example `app.example.com`. Must be a syntactically valid hostname (validated at startup since v0.12.0): dot-separated alphanumeric/`-` labels, up to 253 ASCII characters. Wildcards, underscores, and IPv6 literals are rejected; IPv4 literals are accepted. |
| `reverse_proxy` | Yes | None | List of routing rules for this app. |
| `tls` | No | None | TLS settings for this app. If omitted, the app serves plaintext HTTP only. |

## TLS Options

These options live under `apps.<app_name>.tls`.

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `tls_cert_path` | Yes for static TLS | None | PEM certificate path for this app when using static certificates. |
| `tls_cert_key_path` | Yes for static TLS | None | PEM private key path for this app. The key must be in PKCS8 format. |
| `https_redirection` | No | `true` when both `listen_port` and `listen_port_tls` are set | Per-app HTTP-to-HTTPS redirect switch. If you serve HTTPS only, do not set this option. |
| `client_ca_cert_path` | No | None | CA certificate path for mTLS client authentication. Cleartext requests are never forwarded to an app with this option: they receive a `301` redirect when `https_redirection` is enabled (the default), or `421` when it is explicitly disabled. See [Client Authentication](/docs/guide/advanced/client_auth). |
| `acme` | No | `false` | If `true`, certificates are issued and renewed automatically through ACME instead of `tls_cert_path` and `tls_cert_key_path`. See [ACME (Let's Encrypt) Integration](/docs/guide/advanced/acme). |

## Reverse Proxy Rules

Each app can contain one or more `[[apps.<app_name>.reverse_proxy]]` entries.

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `path` | No | None | Path prefix to match, such as `"/api"` or `"/static"`. Longest-prefix match wins. |
| `replace_path` | No | Preserve original path | Rewritten prefix forwarded upstream. |
| `upstream` | Yes | None | List of backend destinations. |
| `load_balance` | No | `none` | Backend selection strategy. Supported values are `none`, `round_robin`, `random`, `sticky`, and `primary_backup`. Using `sticky` requires the global `sticky_cookie_secret` option. |
| `upstream_options` | No | None | List of request-forwarding behaviors. See [Upstream Options](/docs/guide/advanced/upstream_options). |
| `health_check` | No | None | Active health check configuration. Set `true` for TCP check with defaults, or use a table for full configuration. See [Active Health Check](/docs/guide/advanced/health_check). |

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

## Health Check Options

These options live under `apps.<app_name>.reverse_proxy.health_check`. Alternatively, set `health_check = true` for a TCP check with all defaults. See [Active Health Check](/docs/guide/advanced/health_check) for detailed usage.

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `type` | No | `"tcp"` | Check type: `"tcp"` or `"http"`. |
| `interval` | No | `10` | Seconds between health check probes. |
| `timeout` | No | `5` | Timeout in seconds per check attempt. Must be less than `interval`. |
| `unhealthy_threshold` | No | `3` | Consecutive failures before marking an upstream unhealthy. |
| `healthy_threshold` | No | `2` | Consecutive successes before marking an upstream healthy again. |
| `path` | Yes for `"http"` | None | HTTP check endpoint path. Must start with `/`. |
| `expected_status` | No | `200` | Expected HTTP status code for HTTP checks. |

## Experimental Settings

The `[experimental]` table controls optional and advanced features.

### `[experimental]`

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `ignore_sni_consistency` | No | `false` | If `true`, relax the consistency check between the TLS SNI and the request `Host` header. Keeping it `false` is recommended. This relaxation never applies to apps with client authentication (`client_ca_cert_path`); such requests over a TLS session established for a different server name are always rejected. |
| `connection_handling_timeout` | No | `0` | Total connection handling timeout in seconds. `0` means no timeout. |

### `[experimental.h3]`

Add this table to enable HTTP/3 support. See [HTTP/3](/docs/guide/advanced/http3).

The HTTP/3 connection limit is independent of the global `max_clients` (which covers HTTP/1.1 and HTTP/2 only). The request body limit is the top-level `request_max_body_size`; the former `experimental.h3.request_max_body_size` key was removed in v0.14.0 and now causes a configuration load error.

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `alt_svc_max_age` | No | `3600` | `Alt-Svc` max-age in seconds. |
| `max_concurrent_connections` | No | `512` | Concurrent HTTP/3 (QUIC) connection limit per configured H3 endpoint/listener, covering handshakes through connection close. `0` rejects every HTTP/3 attempt. |
| `max_concurrent_bidistream` | No | `64` | Bidirectional QUIC stream limit. |
| `max_concurrent_unistream` | No | `64` | Unidirectional QUIC stream limit. |
| `max_idle_timeout` | No | `10` | QUIC idle timeout in seconds. `0` means infinite timeout. |

### `[experimental.cache]`

Add this table to enable the hybrid response cache. See [Caching](/docs/guide/advanced/cache).

| Option | Required | Default | Description |
| --- | --- | --- | --- |
| `cache_dir` | No | `./cache` | Cache directory path, relative to the current working directory. |
| `max_cache_entry` | No | `1000` | Maximum number of cache entries. |
| `max_cache_each_size` | No | `65535` | Maximum size in bytes for a single cacheable response. |
| `max_cache_each_size_on_memory` | No | `65535` (same as `max_cache_each_size`) | Maximum size in bytes for keeping a cached response in memory. Larger cached responses are stored as files. With the defaults, every cacheable object is served from memory; the file tier engages when `max_cache_each_size` is raised beyond this value. Set `0` to always use file cache. Worst-case memory use is `max_cache_entry` × this value. |
| `max_cache_total_size` | No | `1073741824` (1 GiB) | Ceiling on the total bytes retained by the cache across the on-memory and file tiers. Accepts an integer in bytes or a string with a binary suffix such as `"256m"` or `"1g"`. When storing a response would exceed it, least-recently-used entries are evicted. Set `"unlimited"` to disable the ceiling; `0` also means unlimited (not zero capacity), so the string form is recommended to avoid confusion with `max_cache_each_size_on_memory = 0`. |

Cached responses are keyed on the request URI forwarded upstream. A backend that varies content by the original client-facing host, scheme, or URI — for example, multiple virtual hosts flattened onto one upstream via `set_upstream_host` or `default_app` — must emit an appropriate `Vary` header on cacheable responses or mark them non-cacheable.

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
- [Trusted Forwarded Proxies](/docs/guide/advanced/trusted_proxies)
- [Client Authentication](/docs/guide/advanced/client_auth)
- [HTTP/3](/docs/guide/advanced/http3)
- [Caching](/docs/guide/advanced/cache)
- [ACME (Let's Encrypt) Integration](/docs/guide/advanced/acme)
- [Active Health Check](/docs/guide/advanced/health_check)
- [PROXY Protocol](/docs/guide/advanced/proxy_protocol)
