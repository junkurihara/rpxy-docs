---
title: 2. Terminating TLS
type: docs
prev: /docs/guide/basics/1_cleartext
next: /docs/guide/basics/3_routing
weight: 2
sidebar:
  open: true
---

Here we describe how to terminate TLS in `rpxy`. In the following, we show how to specify existing TLS certificates and private keys. We also show how to redirect incoming cleartext HTTP requests to HTTPS.

{{< callout type="info" >}}
The integration with ACME (Let's Encrypt) is described in [the advanced section](/docs/guide/advanced/acme).
{{< /callout >}}

First of all, you need to specify a port `listen_port_tls` listening the HTTPS traffic, separately from HTTP port (`listen_port`) in the configuration file.

```toml:config.toml
listen_port = 80
listen_port_tls = 443
```

## Using Existing TLS Certificates

Suppose that you have a TLS certificate and a private key in PEM format, and that you want to use them for serving HTTPS traffic. Then, serving an HTTPS endpoint can be easily done for your desired application just by specifying TLS certificates and private keys in PEM files.

```toml
listen_port = 80
listen_port_tls = 443

[apps."app_name"]
server_name = 'app1.example.com'
tls = { tls_cert_path = 'server.crt',  tls_cert_key_path = 'server.key' }
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]
```

In the above setting, both cleartext HTTP requests to port 80 and ciphertext HTTPS requests to port 443 are routed to the backend `app1.local:8080` in the same fashion. If you don't need to serve cleartext requests, just remove `listen_port = 80` and specify only `listen_port_tls = 443`.

{{< callout type="info" >}}
We should note that the private key specified by `tls_cert_key_path` must be *in PKCS8 format*. (See [TIPS](/docs/tips/) to convert a PKCS1 formatted private key to PKCS8 one.)
{{< /callout >}}

## Redirecting Cleartext HTTP Requests to HTTPS

In the current Web, we believe it is common to serve everything through HTTPS rather than HTTP, and hence *https redirection* is often used for HTTP requests. When you specify both `listen_port` and `listen_port_tls`, you can enable an option of such  redirection by making `https_redirection` true.

```toml:config.toml
tls = { https_redirection = true, tls_cert_path = 'server.crt', tls_cert_key_path = 'server.key' }
```

If it is true, `rpxy` returns the status code `301` to the cleartext request with new location `https://<requested_host>/<requested_query_and_path>` served over TLS.
