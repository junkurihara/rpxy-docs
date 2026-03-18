---
title: Active Health Check
type: docs
prev: /docs/guide/advanced/proxy_protocol
weight: 7
sidebar:
  open: true
---

## Overview

When `rpxy` forwards requests to multiple upstream backends, it is important to avoid sending traffic to servers that are down or unhealthy. The **active health check** feature periodically probes each upstream and automatically removes unhealthy ones from the load balancing pool.

Key behaviors:

- **TCP** or **HTTP** probe types
- Configurable per `[[apps.<name>.reverse_proxy]]` block
- Upstream is marked unhealthy after a configurable number of consecutive failures and marked healthy again after consecutive successes
- An immediate probe runs at startup, followed by probes at a fixed interval
- If **all** upstreams become unhealthy, `rpxy` logs a warning and continues routing on a best-effort basis

{{< callout type="info" >}}
The active health check requires the `health-check` Cargo feature to be enabled at build time. Pre-built binaries and Docker images with this feature enabled are available from the official releases.
{{< /callout >}}

## Configuration

### Simplest Form — TCP Check with Defaults

Setting `health_check = true` enables a TCP connect check with all default parameters:

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "backend1:8080" },
  { location = "backend2:8080" },
]
health_check = true
```

### Full TCP Configuration

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "backend1:8080" },
  { location = "backend2:8080" },
]

[apps.app1.reverse_proxy.health_check]
type = "tcp"
interval = 10
timeout = 5
unhealthy_threshold = 3
healthy_threshold = 2
```

### HTTP Health Check

For HTTP health checks, `rpxy` sends an HTTP request to a specified path and checks the response status code:

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "backend1:8080" },
  { location = "backend2:8080" },
]

[apps.app1.reverse_proxy.health_check]
type = "http"
path = "/healthz"
expected_status = 200
interval = 10
timeout = 5
unhealthy_threshold = 3
healthy_threshold = 2
```

## Options

| Option | Default | Description |
| --- | --- | --- |
| `type` | `"tcp"` | Check type: `"tcp"` or `"http"`. |
| `interval` | `10` | Seconds between health check probes. |
| `timeout` | `5` | Timeout in seconds per check attempt. Must be less than `interval`. |
| `unhealthy_threshold` | `3` | Consecutive failures before marking an upstream unhealthy. |
| `healthy_threshold` | `2` | Consecutive successes before marking an upstream healthy again. |
| `path` | — | HTTP check endpoint path. **Required** when `type = "http"`. Must start with `/`. |
| `expected_status` | `200` | Expected HTTP status code for HTTP checks. |

## Interaction with Load Balancing

The health check feature works together with the `load_balance` option:

| `load_balance` | Behavior with Health Check |
| --- | --- |
| `"none"` | Without health check, always picks the first upstream. With health check, picks the **first healthy** upstream. |
| `"round_robin"` | Skips unhealthy upstreams in the rotation. |
| `"random"` | Selects randomly from healthy upstreams only. |
| `"sticky"` | Falls back to another healthy upstream when the sticky target is unhealthy. |
| `"primary_backup"` | Always routes to the first healthy upstream. **Requires** `health_check` to be enabled. |

{{< callout type="warning" >}}
The `"primary_backup"` load balance mode requires `health_check` to be configured. `rpxy` will return an error at startup if `load_balance = "primary_backup"` is set without a `health_check` configuration.
{{< /callout >}}

## Example: Primary/Backup with HTTP Health Check

A common pattern is to have a primary backend with a backup that only receives traffic when the primary is down:

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "primary.internal:8080" },
  { location = "backup.internal:8080" },
]
load_balance = "primary_backup"

[apps.app1.reverse_proxy.health_check]
type = "http"
path = "/healthz"
interval = 5
unhealthy_threshold = 2
healthy_threshold = 1
```

In this configuration, `rpxy` always sends traffic to `primary.internal:8080` as long as it is healthy. If the primary fails two consecutive health checks, traffic switches to `backup.internal:8080`. Once the primary passes one health check again, traffic returns to it.
