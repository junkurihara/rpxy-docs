---
title: 1. Cleartext HTTP Reverse Proxy
type: docs
prev: /docs/guide/basics/
next: /docs/guide/basics/2_tls
weight: 1
sidebar:
  open: true
---

Here we describe how to configure a simple reverse proxy server that forwards incoming *cleartext* HTTP requests to a backend application.

## The Simplest Configuration

The simplest configuration of the TOML-based configuration file, say `config.toml`, is given like the following.

```toml:config.toml
listen_port = 80

[apps.app1]
server_name = 'app1.example.com'
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]
```

In the above setting, `rpxy` listens on port 80 (TCP) and serves incoming *cleartext* HTTP request including a `app1.example.com` in its HOST header or URL in its Request line, e.g., following HTTP request messages.

```plaintext
GET http://app1.example.com/path/to HTTP/1.1\r\n
```

or

```plaintext
GET /path/to HTTP/1.1\r\n
HOST: app1.example.com\r\n
```

Otherwise, say, a request to `other.example.com` is simply rejected with the status code `40x`.

Also in the above setting, the outgoing connection to the backend application is over *cleartext* HTTP, not HTTPS, as well as the incoming connection. If you need to forward the request to the backend application over HTTPS, see the following subsection.

## Serving Multiple Domain Names

If you want to host multiple and distinct domain names in a single IP address/port, simply create multiple `app."<app_name>"` entries in the config file like the following.

```toml:config.toml
listen_port = 80

default_app = "app1"

[apps.app1]
server_name = "app1.example.com"
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]

[apps.app2]
server_name = "app2.example.org"
reverse_proxy = [{ upstream = [{ location = 'app2.local:8888' }] }]
```

In the above setting, by specifying `default_app` entry, any *cleartext HTTP* request will be served by the specified application if its HOST header or URL in Request line doesn't match any `server_name`s in `reverse_proxy` entries.

{{< callout type="info" >}}
Any *HTTPS* request with no matched `server_name` will be rejected since the secure connection cannot be established for the unknown server name, i.e., Common Name of the server certificate.
{{< /callout >}}

## Upstream Connection over HTTPS to Backend Application

In the above example, request messages are routed to backend applications over cleartext HTTP. If a backend channel to an app needs to established over TLS, e.g., requests forwarded to `https://app1.localdomain:8080`, you need to enable a `tls` option for the `location` requiring HTTPS connection.

```toml:config.toml
[apps.app_backend_https]
server_name = "app_backend_https.example.com"
revese_proxy = [
  { location = 'app1.localdomain:8080', tls = true }
]
```

## Load Balancing with Multiple Backend Locations

You can specify multiple backend locations in the `reverse_proxy` array for *load-balancing* with an appropriate `load_balance` option. Currently it works with the following options:

- `round_robin`: for each request, one of the backend locations is chosen in a round-robin fashion;
- `random`: for each request, one of the backend locations is chosen randomly;
- `sticky`: a backend location is chosen as `round_robin` but the *session-persistance* is guaranteed using cookie.

If `load_balance` is not specified, the first backend location is always chosen.

```toml
[apps."app_name"]
server_name = 'app1.example.com'
reverse_proxy = [
  { location = 'app1.local:8080' },
  { location = 'app2.local:8000' }
]
load_balance = 'round_robin' # or 'random' or 'sticky'
```

{{< callout type="info" >}}
Currently, no health-checking mechanism is implemented. If a backend location is down, the request to the location will be failed.
{{< /callout >}}
