---
title: rpxy-l4
type: docs
prev: /docs/
weight: 35
---

[`rpxy-l4`](https://github.com/junkurihara/rust-rpxy-l4) is a sister project of `rpxy`. While `rpxy` focuses on Layer 7 HTTP(S) reverse proxying, `rpxy-l4` targets Layer 4 proxying for TCP and UDP workloads.

{{< callout type="warning" >}}
While `rpxy-l4` is already working in some real environments, it is an experimental project driven by the author's research interests. For now, treat this page as an overview and use the upstream repository for detailed setup and operational guidance.
{{< /callout >}}

## What `rpxy-l4` is for

`rpxy-l4` is intended for cases where you need to proxy or demultiplex traffic before HTTP exists, or where the traffic is not HTTP at all.

Examples include:

- TCP and UDP forwarding
- Protocol multiplexing on the same port
- SNI and ALPN based forwarding for TLS or QUIC
- Preserving client IP information with HAProxy PROXY protocol

## When to use which project

| If you need... | Use |
| --- | --- |
| HTTP/1.1, HTTP/2, HTTP/3 reverse proxying, TLS termination, ACME, path-based routing | `rpxy` |
| TCP/UDP proxying, protocol-level demultiplexing, or an L4 proxy in front of `rpxy` | `rpxy-l4` |
| HTTP routing plus client IP preservation or L4 traffic steering | `rpxy-l4` in front of `rpxy` |

## Using `rpxy-l4` together with `rpxy`

One common deployment is:

```plaintext
Client --> rpxy-l4 --> rpxy --> Backend App
```

In that arrangement, `rpxy-l4` handles Layer 4 concerns such as TCP forwarding or PROXY protocol, and `rpxy` continues to handle HTTP routing, TLS termination, ACME, caching, and other Layer 7 features.

## Where to find the full documentation

At the moment, the primary reference for [`rpxy-l4`](https://github.com/junkurihara/rust-rpxy-l4) is its upstream repository:

- GitHub: [junkurihara/rust-rpxy-l4](https://github.com/junkurihara/rust-rpxy-l4)

For the integration point most relevant to `rpxy`, see [PROXY Protocol](/docs/guide/advanced/proxy_protocol).
