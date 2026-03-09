---
title: ACME (Let's Encrypt)連携
type: docs
prev: /docs/guide/advanced/cache
weight: 3
sidebar:
  open: true
---

[証明書と秘密鍵の静的ファイルを使用する方法](/docs/guide/basics/2_tls)に加えて、証明書の自動発行と更新、つまり[RFC8555](https://www.rfc-editor.org/rfc/rfc8555)として標準化されたACME (Automated Certificate Management Environment)が`rpxy`で利用できます。この機能を有効にするには、設定ファイルでACMEが必要な各アプリケーションに以下のエントリを指定します。

```toml:config.toml
# TLSポート。ACMEチャレンジにも使用されます。
listen_port_tls = 443

# ACME有効なドメイン名。
# acmeオプションはexperimentalセクションで指定する必要があります。
[apps."app_with_acme"]
server_name = 'example.org'
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]
tls = { https_redirection = true, acme = true } # tls_cert_pathやtls_cert_key_pathは指定しないでください
```

ACMEは、ACME [`TLS-ALPN-01` (RFC8737)](https://www.rfc-editor.org/rfc/rfc8737)プロトコルを使用して`server_name`の証明書を取得します。したがって、TLSポート443を公開するだけで済みます。また、この場合、アプリケーションの`tls_cert_path`と`tls_cert_key_path`を指定する必要はありません。

ACME有効なすべてのドメインに対して、証明書と秘密鍵を取得するために以下の設定が参照されます。

```toml:config.toml
# グローバルACME設定。指定しない場合、ACMEは無効です。
[experimental.acme]
# ACME登録用のメールアドレス。
email = "test@example.com"
# オプション: ACMEディレクトリURL。[デフォルト: "https://acme-v02.api.letsencrypt.org/directory"]
dir_url = "https://acme-staging-v02.api.letsencrypt.org/directory"
# オプション: 取得した証明書と秘密鍵を保存するディレクトリ（現在の作業ディレクトリからの相対パス）。[デフォルト: "./acme_registry"]
registry_path = "./acme_registry"
```

上記の設定はすべてのACME有効ドメインに共通です。ドメインの所有権を証明するために、HTTPSポートを公開する必要があります。
