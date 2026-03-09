---
title: TLSクライアント認証
type: docs
prev: /docs/guide/advanced/http3
next: /docs/guide/advanced/acme
weight: 2
sidebar:
  open: true
---

TLSクライアント認証は、`"app_name"`で指定されたドメインに対して`apps."app_name".tls.client_ca_cert_path`を設定することで有効になります。

```toml
[apps.localhost]
server_name = 'localhost' # Domain name
tls = { https_redirection = true, tls_cert_path = './server.crt', tls_cert_key_path = './server.key', client_ca_cert_path = './client_cert.ca.crt' }
```

{{< callout type="warning" >}}
現在、クライアント認証を有効にしたアプリケーションでのHTTP/3サポートには制限があります。アプリケーションにクライアント認証が設定されている場合、そのアプリケーションではHTTP/3は動作しません。
{{< /callout >}}
