---
title: TLS Client Authentication
type: docs
prev: /docs/guide/advanced/http3
next: /docs/guide/advanced/acme
weight: 2
sidebar:
  open: true
---

TLS client authentication is enabled when `apps."app_name".tls.client_ca_cert_path` is set for the domain specified by `"app_name"` like

```toml
[apps.localhost]
server_name = 'localhost' # Domain name
tls = { https_redirection = true, tls_cert_path = './server.crt', tls_cert_key_path = './server.key', client_ca_cert_path = './client_cert.ca.crt' }
```

{{< callout type="warning" >}}
Currently we have a limitation on HTTP/3 support for applications that enables client authentication. If an application is set with client authentication, HTTP/3 doesn't work for the application.
{{< /callout >}}

## Security Behaviors

Client-authentication apps are subject to the following hardening behaviors:

- **Cleartext requests are never forwarded to a client-authentication app.** With `https_redirection` enabled (the default), cleartext requests receive the normal `301` redirect. If `https_redirection = false` is explicitly set, cleartext requests resolved to the app — including through the plaintext `default_app` fallback — fail closed with status code `421` before any upstream contact. (Since v0.14.0)
- **The `ignore_sni_consistency` relaxation never applies.** A request that reaches a client-authentication app over a TLS session established for a different server name is always rejected before forwarding, regardless of the `experimental.ignore_sni_consistency` setting. (Since v0.13.3)
- **TLS sessions are never resumed for client-authentication apps.** Client-certificate verification runs on every mTLS connection, matching the industry practice of disabling session resumption for mutual TLS. (Since v0.13.1)
- **Handshake failures are audit-logged.** TLS handshake failures, including client-certificate verification failures, are logged as structured records carrying the source IP, the SNI, and a stable failure category such as `client_cert`. (Since v0.12.0)
