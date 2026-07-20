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

## セキュリティ上の挙動

クライアント認証を有効にしたアプリケーションには、次のような保護的挙動が適用されます。

- **平文リクエストはクライアント認証アプリに決して転送されません。**`https_redirection`が有効（デフォルト）の場合、平文リクエストには通常どおり`301`リダイレクトが返ります。`https_redirection = false`を明示的に設定した場合、そのアプリに解決された平文リクエストは、平文の`default_app`フォールバック経由を含めて、upstreamに接続する前にステータスコード`421`で拒否されます。（v0.14.0以降）
- **`ignore_sni_consistency`の緩和は適用されません。**別のserver nameで確立されたTLSセッション経由でクライアント認証アプリに到達したリクエストは、`experimental.ignore_sni_consistency`の設定にかかわらず、転送前に常に拒否されます。（v0.13.3以降）
- **クライアント認証アプリではTLSセッションは再開されません。**mTLS接続のたびにクライアント証明書の検証が実行されます。これはmutual TLSでセッション再開を無効化する業界慣行に沿った挙動です。（v0.13.1以降）
- **ハンドシェイク失敗は監査ログに記録されます。**クライアント証明書の検証失敗を含むTLSハンドシェイクの失敗は、接続元IP、SNI、`client_cert`などの失敗カテゴリを含む構造化レコードとしてログに記録されます。（v0.12.0以降）
