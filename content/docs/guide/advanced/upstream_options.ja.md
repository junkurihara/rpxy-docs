---
title: アップストリームオプション
type: docs
prev: /docs/guide/advanced/cache
next: /docs/guide/advanced/post_quantum
weight: 45
sidebar:
  open: true
---

`rpxy`は、アップストリームのバックエンドアプリケーションへのリクエストメッセージの転送方法を制御するためのいくつかのオプションを提供しています。これらはリバースプロキシエントリごとに`upstream_options`配列で指定します。

## 設定

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = 'backend.local:8080' },
]
upstream_options = [
  "set_upstream_host",
  "forwarded_header",
]
```

## 利用可能なオプション

### `keep_original_host`

クライアントリクエストの元の`Host`ヘッダーを保持します。これが**デフォルトの動作**であり、`set_upstream_host`と両方指定された場合はこちらが優先されます。

### `set_upstream_host`

`Host`ヘッダーをアップストリームのホスト名（例: `backend.local:8080`）で上書きします。`keep_original_host`も指定されている場合は無視されます。

### `upgrade_insecure_requests`

転送リクエストに`Upgrade-Insecure-Requests: 1`ヘッダーが存在しない場合に追加します。

### `force_http11_upstream`

アップストリーム接続にHTTP/1.1を強制します。`force_http2_upstream`と排他的です。

### `force_http2_upstream`

アップストリーム接続にHTTP/2を強制します。`force_http11_upstream`と排他的です。

### `forwarded_header`

標準の[`Forwarded`ヘッダー (RFC 7239)](https://www.rfc-editor.org/rfc/rfc7239)を生成して転送リクエストに追加します。

デフォルトでは、`rpxy`は設定なしで以下の転送ヘッダーをすべてのリクエストに自動的に追加します:

| ヘッダー | 説明 |
| --- | --- |
| `X-Forwarded-For` | クライアントのIPアドレスが追加されます。既に存在する場合、新しいIPはカンマ区切りで追加されます。 |
| `X-Forwarded-Proto` | 受信接続に基づいて`https`または`http`に設定されます。既に存在しない場合のみ追加されます。 |
| `X-Forwarded-Port` | リッスンポートに設定されます。既に存在しない場合のみ追加されます。 |
| `X-Forwarded-SSL` | 受信接続に基づいて`on`または`off`に設定されます。 |
| `X-Real-IP` | クライアントのIPアドレスに設定されます。 |
| `X-Original-URI` | 元のリクエストURIに設定されます。 |

`forwarded_header`オプションは、これらのデフォルトヘッダー**に加えて**RFC 7239の`Forwarded`ヘッダーを追加します:

```plaintext
Forwarded: for=192.0.2.1;proto=https;host=app1.example.com
```

{{< callout type="info" >}}
受信リクエストに既に`Forwarded`ヘッダーが含まれている場合、`rpxy`は`forwarded_header`オプションがなくても整合性のために更新します。このオプションは、`Forwarded`ヘッダーがまだ存在しない場合に**新しく生成する**かどうかを制御します。
{{< /callout >}}

## 例

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = 'www.example.com', tls = true },
  { location = 'www.example.org', tls = true },
]
load_balance = "round_robin"
upstream_options = [
  "set_upstream_host",
  "forwarded_header",
  "force_http11_upstream",
]

[[apps.app1.reverse_proxy]]
path = '/api'
replace_path = '/'
upstream = [
  { location = 'api.internal:3000' },
]
upstream_options = [
  "upgrade_insecure_requests",
]
```
