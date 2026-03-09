---
title: PROXY Protocol
type: docs
prev: /docs/guide/advanced/post_quantum
weight: 6
sidebar:
  open: true
---

## What is PROXY Protocol?

When a reverse proxy or load balancer sits between a client and a backend server, the backend server sees the proxy's IP address as the source, not the original client's. The [PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt) solves this problem at the **TCP layer** by having the upstream proxy prepend a small header to each TCP connection that carries the original client's IP address and port.

Unlike HTTP-level solutions such as `X-Forwarded-For` or `Forwarded` headers, PROXY protocol works transparently for any TCP-based protocol and cannot be spoofed by the application-level client.

## How It Works with rpxy

`rpxy` supports receiving PROXY protocol v1/v2 headers on **incoming** TCP connections. This means `rpxy` can be placed behind an L4 load balancer or proxy (such as AWS ELB, HAProxy, Nginx, or [rpxy-l4](https://github.com/junkurihara/rpxy-l4)) that speaks PROXY protocol, and `rpxy` will recover the original client's source IP and port from the PROXY header.

The typical deployment looks like:

```plaintext
Client --> L4 Load Balancer (PROXY protocol) --> rpxy --> Backend App
```

Without PROXY protocol, `rpxy` would only see the load balancer's IP as the client address. With PROXY protocol enabled, `rpxy` parses the prepended header and uses the real client IP for logging and forwarding.

## Configuration

PROXY protocol support is included in the default build. To enable it, add the `[experimental.tcp_recv_proxy_protocol]` section to your configuration file:

```toml
[experimental.tcp_recv_proxy_protocol]
trusted_proxies = ["127.0.0.1/32", "10.0.0.0/8"]  # required, non-empty CIDR list (IPv4 and/or IPv6)
timeout = 50  # optional, milliseconds. Default: 50ms. 0 = fallback to 5s.
```

| Parameter | Description |
| --- | --- |
| `trusted_proxies` | **Required**. Non-empty list of CIDR ranges specifying trusted proxy sources. Example: `["127.0.0.1/32", "::1/128"]` |
| `timeout` | Optional. Timeout in milliseconds for receiving the PROXY header. Default is 50ms. Setting to 0 falls back to 5s (not recommended). |

When enabled, `rpxy` expects all incoming TCP connections from trusted proxies to include a PROXY protocol header and will extract the original client address information from it.

{{< callout type="error" >}}
**Security Warning**: PROXY headers are not authenticated. When configured, **all TCP connections must originate from a listed trusted proxy**. Restrict access with firewall rules to prevent spoofing of client addresses.
{{< /callout >}}

## Limitations

{{< callout type="warning" >}}
**HTTP/3 (QUIC) is not supported** for PROXY protocol since QUIC's underlying UDP is connectionless. PROXY protocol support applies only to HTTP/1.1 and HTTP/2 over TCP.
{{< /callout >}}

## Use Cases

- Running `rpxy` behind **AWS Elastic Load Balancer (ELB)**
- Running `rpxy` behind **HAProxy** with PROXY protocol enabled
- Running `rpxy` behind **Nginx** configured as a TCP proxy with PROXY protocol
- Chaining with **rpxy-l4** for Layer 4 load balancing
