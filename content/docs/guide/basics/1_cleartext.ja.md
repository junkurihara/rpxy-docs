---
title: 1. 平文HTTPリバースプロキシ
type: docs
prev: /docs/guide/basics/
next: /docs/guide/basics/2_tls
weight: 1
sidebar:
  open: true
---

ここでは、受信した*平文*HTTPリクエストをバックエンドアプリケーションに転送するシンプルなリバースプロキシサーバーの設定方法を説明します。

## 最もシンプルな設定

TOML形式の設定ファイル（例: `config.toml`）の最もシンプルな設定は以下の通りです。

```toml:config.toml
listen_port = 80

[apps.app1]
server_name = 'app1.example.com'
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]
```

上記の設定では、`rpxy`はポート80 (TCP)でリッスンし、HOSTヘッダーまたはリクエスト行のURLに`app1.example.com`を含む*平文*HTTPリクエストを処理します。例えば、以下のようなHTTPリクエストメッセージです。

```plaintext
GET http://app1.example.com/path/to HTTP/1.1\r\n
```

または

```plaintext
GET /path/to HTTP/1.1\r\n
HOST: app1.example.com\r\n
```

それ以外のリクエスト、例えば`other.example.com`へのリクエストは、ステータスコード`40x`で拒否されます。

また上記の設定では、受信接続と同様にバックエンドアプリケーションへの送信接続も*平文*HTTPで行われ、HTTPSではありません。バックエンドアプリケーションへのリクエストをHTTPSで転送する必要がある場合は、以下のサブセクションを参照してください。

## 複数ドメイン名の提供

単一のIPアドレス/ポートで複数の異なるドメイン名をホストしたい場合は、設定ファイルに複数の`app."<app_name>"`エントリを作成します。

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

上記の設定では、`default_app`エントリを指定することにより、HOSTヘッダーまたはリクエスト行のURLが`reverse_proxy`エントリの`server_name`のいずれにも一致しない場合、指定されたアプリケーションが*平文HTTP*リクエストを処理します。

{{< callout type="info" >}}
`server_name`に一致しない*HTTPS*リクエストは拒否されます。これは、不明なサーバー名（サーバー証明書のCommon Name）に対してセキュア接続を確立できないためです。
{{< /callout >}}

## バックエンドアプリケーションへのHTTPS接続

上記の例では、リクエストメッセージは平文HTTPでバックエンドアプリケーションにルーティングされます。アプリへのバックエンドチャネルをTLS経由で確立する必要がある場合（例: `https://app1.localdomain:8080`へのリクエスト転送）、HTTPS接続が必要な`location`に対して`tls`オプションを有効にする必要があります。

```toml:config.toml
[apps.app_backend_https]
server_name = "app_backend_https.example.com"
reverse_proxy = [
  { location = 'app1.localdomain:8080', tls = true }
]
```

## 複数バックエンドによるロードバランシング

適切な`load_balance`オプションを指定して、`reverse_proxy`配列に複数のバックエンドロケーションを指定することで*ロードバランシング*が可能です。現在、以下のオプションが利用できます:

- `round_robin`: リクエストごとに、バックエンドロケーションがラウンドロビン方式で選択されます;
- `random`: リクエストごとに、バックエンドロケーションがランダムに選択されます;
- `sticky`: `round_robin`と同様にバックエンドロケーションが選択されますが、Cookieを使用した*セッション永続性*が保証されます。

`load_balance`が指定されていない場合、最初のバックエンドロケーションが常に選択されます。

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
現在、ヘルスチェック機能は実装されていません。バックエンドロケーションがダウンしている場合、そのロケーションへのリクエストは失敗します。
{{< /callout >}}
