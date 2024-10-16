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
