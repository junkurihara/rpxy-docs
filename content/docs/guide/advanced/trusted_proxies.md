---
title: Trusted Forwarded Proxies
type: docs
prev: /docs/guide/advanced/upstream_options
next: /docs/guide/advanced/post_quantum
weight: 6
sidebar:
  open: true
---

## Overview

When `rpxy` runs behind another load balancer, reverse proxy, or CDN, the original client information (IP address, protocol scheme) arrives through forwarding headers such as `X-Forwarded-For` and `Forwarded`. These headers are trivially spoofable by clients, so they must be trusted only when they come from a proxy you actually control or trust.

Since v0.12.0, **no proxy is trusted by default**: `rpxy` ignores incoming forwarding headers and rebuilds `X-Forwarded-For`, `Forwarded`, and related headers from the immediate peer address only. The global `trusted_forwarded_proxies` option restores the chain for deployments with a trusted front proxy.

## Configuration

```toml
# CIDR blocks of your front proxies
trusted_forwarded_proxies = ["10.0.0.0/8", "192.168.0.0/16"]

# Built-in provider aliases can be mixed with CIDRs
trusted_forwarded_proxies = ["cloudflare", "10.0.0.0/8"]

# Trust all IPv4 proxies (NOT recommended)
trusted_forwarded_proxies = "0.0.0.0/0"
# For full dual-stack trust-all, specify both "0.0.0.0/0" and "::/0".
```

When the immediate peer of an incoming connection is within the trusted ranges, `rpxy` preserves and normalizes the forwarding information learned through the trusted proxies, rewrites the outgoing `X-Forwarded-For` and related headers from that normalized chain, and falls back safely to the immediate-peer-only view when the incoming forwarding view is malformed or inconsistent.

### Built-in Provider Aliases

The aliases `"cloudflare"`, `"fastly"`, and `"cloudfront"` expand to the corresponding provider's published IP ranges. The ranges are built-in static snapshots resolved at configuration load; `rpxy` never fetches provider IP lists at runtime, so startup does not depend on external network reachability. The snapshots can be refreshed explicitly with the `rpxy-trusted-proxies` updater in the [rust-rpxy repository](https://github.com/junkurihara/rust-rpxy/tree/main/rpxy-trusted-proxies).

## Operator Requirement

{{< callout type="warning" >}}
Any proxy listed in `trusted_forwarded_proxies` must **overwrite or normalize** the incoming `X-Forwarded-Proto` (and related headers) rather than appending client-supplied values — for example, Nginx `proxy_set_header X-Forwarded-Proto $scheme;`. Otherwise an attacker upstream of the trusted proxy can spoof the forwarded scheme or chain. AWS ALB and CloudFront satisfy this by default.
{{< /callout >}}

## Interaction with Sticky Sessions

The `Secure` attribute of the sticky-session cookie (`load_balance = "sticky"`) is applied when the client-visible request scheme is HTTPS. When `rpxy` itself terminates TLS this is automatic. When `rpxy` runs behind an external TLS terminator (ALB, CloudFront, Nginx, HAProxy, etc.), the terminator's address must be listed in `trusted_forwarded_proxies` for `Secure` to be applied, since `rpxy` honors `X-Forwarded-Proto: https` (or `Forwarded: proto=https`) only from trusted peers.

## Relation to PROXY Protocol

`trusted_forwarded_proxies` governs trust in **HTTP-level** forwarding headers. To recover the original client address at the **TCP level** from an L4 proxy, use the [PROXY protocol](/docs/guide/advanced/proxy_protocol) support (`[experimental.tcp_recv_proxy_protocol]`), which has its own independent `trusted_proxies` list.
