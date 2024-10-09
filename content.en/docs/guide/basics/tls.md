---
title: 2. Terminating TLS
type: docs
prev: /docs/guide/basics/cleartext
next: /docs/guide/basics/routing
weight: 2
sidebar:
  open: true
---

Here we describe how to terminate TLS in `rpxy`. In the following, we show how to specify existing TLS certificates and private keys, and how to integrate with ACME (Let's Encrypt) for automatic certificate issuance. We also show how to redirect incoming cleartext HTTP requests to HTTPS.

First of all, you need to specify a port `listen_port_tls` listening the HTTPS traffic, separately from HTTP port (`listen_port`) in the configuration file.

```toml:config.toml
listen_port = 80
listen_port_tls = 443
```

## Specifying Existing TLS Certificates

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

## ACME (Let's Encrypt) Integration

In addition to use static files of certificates and private keys, the automatic issuance and renewal of certificates, i.e., [ACME (Automated Certificate Management Environment; RFC8555)](https://www.rfc-editor.org/rfc/rfc8555), are available in `rpxy`. To enable this feature, you need to specify the following entries for each application requiring ACME in the configuration file.

```toml:config.toml
# TLS port, which is also used for ACME challenge.
listen_port_tls = 443

# ACME enabled domain name.
# Note that acme option must be specified in the experimental section.
[apps."app_with_acme"]
server_name = 'example.org'
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]
tls = { https_redirection = true, acme = true } # do not specify tls_cert_path and/or tls_cert_key_path
```

ACME will be used to get a certificate for the `server_name` with ACME [`TLS-ALPN-01` (RFC8737)](https://www.rfc-editor.org/rfc/rfc8737) protocol. So all you need is to open
the TLS port 443 to the public. Also in this case, you don't need to specify `tls_cert_path` and `tls_cert_key_path` for the application.

For every ACME enabled domain, the following settings are referred to acquire a certificate and a private key.

```toml:config.toml
# Global ACME settings. Unless specified, ACME is disabled.
[experimental.acme]
# Email address for ACME registration.
email = "test@example.com"
# Optional: ACME directory URL. [default: "https://acme-v02.api.letsencrypt.org/directory"]
dir_url = "https://acme-staging-v02.api.letsencrypt.org/directory"
# Optional: Directory storing retrieved certificates and private keys, which is relative to the current working directory. [default: "./acme_registry"]
registry_path = "./acme_registry"
```

The above configuration is common to all ACME enabled domains. Note that the https port must be open to the public to prove the domain ownership.

{{< callout type="warning" >}}
This is a brand-new feature and maybe still unstable.
{{< /callout >}}

## Redirecting Cleartext HTTP Requests to HTTPS

In the current Web, we believe it is common to serve everything through HTTPS rather than HTTP, and hence *https redirection* is often used for HTTP requests. When you specify both `listen_port` and `listen_port_tls`, you can enable an option of such  redirection by making `https_redirection` true.

```toml:config.toml
tls = { https_redirection = true, tls_cert_path = 'server.crt', tls_cert_key_path = 'server.key' }
```

If it is true, `rpxy` returns the status code `301` to the cleartext request with new location `https://<requested_host>/<requested_query_and_path>` served over TLS.
