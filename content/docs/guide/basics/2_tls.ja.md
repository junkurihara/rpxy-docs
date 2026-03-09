---
title: 2. TLS終端
type: docs
prev: /docs/guide/basics/1_cleartext
next: /docs/guide/basics/3_routing
weight: 2
sidebar:
  open: true
---

ここでは`rpxy`でのTLS終端の方法を説明します。既存のTLS証明書と秘密鍵の指定方法、および平文HTTPリクエストをHTTPSにリダイレクトする方法を示します。

{{< callout type="info" >}}
ACME (Let's Encrypt)との連携については[高度な使い方セクション](/docs/guide/advanced/acme)で説明しています。
{{< /callout >}}

まず、設定ファイルでHTTPポート（`listen_port`）とは別に、HTTPSトラフィックをリッスンするポート`listen_port_tls`を指定する必要があります。

```toml:config.toml
listen_port = 80
listen_port_tls = 443
```

## 既存のTLS証明書の使用

PEM形式のTLS証明書と秘密鍵があり、HTTPSトラフィックの提供に使用したい場合、PEMファイルでTLS証明書と秘密鍵を指定するだけで、希望するアプリケーションのHTTPSエンドポイントを簡単に設定できます。

```toml
listen_port = 80
listen_port_tls = 443

[apps."app_name"]
server_name = 'app1.example.com'
tls = { tls_cert_path = 'server.crt',  tls_cert_key_path = 'server.key' }
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]
```

上記の設定では、ポート80への平文HTTPリクエストとポート443への暗号化HTTPSリクエストの両方が、同じ方法でバックエンド`app1.local:8080`にルーティングされます。平文リクエストを提供する必要がない場合は、`listen_port = 80`を削除して`listen_port_tls = 443`のみを指定してください。

{{< callout type="info" >}}
`tls_cert_key_path`で指定する秘密鍵は*PKCS8形式*である必要があります。（PKCS1形式の秘密鍵をPKCS8に変換するには[TIPS](/docs/tips/)を参照してください。）
{{< /callout >}}

## 平文HTTPリクエストのHTTPSリダイレクト

現在のWebでは、HTTPではなくHTTPSですべてを提供するのが一般的であり、HTTPリクエストに対する*HTTPSリダイレクト*がよく使用されます。`listen_port`と`listen_port_tls`の両方を指定している場合、`https_redirection`をtrueにすることでリダイレクトオプションを有効にできます。

```toml:config.toml
tls = { https_redirection = true, tls_cert_path = 'server.crt', tls_cert_key_path = 'server.key' }
```

trueの場合、`rpxy`は平文リクエストに対してステータスコード`301`を返し、TLS経由で提供される新しいロケーション`https://<requested_host>/<requested_query_and_path>`にリダイレクトします。
