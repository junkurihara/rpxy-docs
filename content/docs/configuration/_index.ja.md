---
title: 設定オプション
type: docs
prev: /docs/
weight: 30
---

このページは、[`config-example.toml`](https://github.com/junkurihara/rust-rpxy/blob/main/config-example.toml) に出てくる設定キーのクイックリファレンスです。
セットアップ手順は [基本設定](/docs/guide/basics) と [高度な使い方](/docs/guide/advanced) を参照してください。

{{< callout type="info" >}}
このページは辞書的に引けることを重視しています。具体的な挙動や構成例は、末尾の関連ガイドを参照してください。
{{< /callout >}}

## クイックナビゲーション

- [クイックナビゲーション](#クイックナビゲーション)
- [最小構成例](#最小構成例)
- [グローバル設定](#グローバル設定)
- [アプリケーション定義](#アプリケーション定義)
  - [`[apps.<app_name>]`](#appsapp_name)
- [TLS オプション](#tls-オプション)
- [リバースプロキシ設定](#リバースプロキシ設定)
- [Upstream エントリ](#upstream-エントリ)
- [`upstream_options` の値](#upstream_options-の値)
- [ヘルスチェックオプション](#ヘルスチェックオプション)
- [Experimental 設定](#experimental-設定)
  - [`[experimental]`](#experimental)
  - [`[experimental.h3]`](#experimentalh3)
  - [`[experimental.cache]`](#experimentalcache)
  - [`[experimental.acme]`](#experimentalacme)
  - [`[experimental.tcp_recv_proxy_protocol]`](#experimentaltcp_recv_proxy_protocol)
- [関連ガイド](#関連ガイド)

## 最小構成例

```toml
listen_port = 80
listen_port_tls = 443

[apps.app1]
server_name = "app1.example.com"
tls = { tls_cert_path = "./server.crt", tls_cert_key_path = "./server.key" }

[[apps.app1.reverse_proxy]]
upstream = [
  { location = "app1.local:8080" },
]
```

## グローバル設定

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `listen_port` | いいえ | なし | 平文 HTTP を待ち受ける TCP ポートです。`listen_port` と `listen_port_tls` の少なくとも一方は必須です。 |
| `listen_port_tls` | いいえ | なし | HTTPS/TLS を待ち受ける TCP ポートです。TLS を使う場合、ACME を使う場合、HTTP/3 を有効にする場合に必要です。 |
| `public_https_port` | いいえ | TLS 有効時は `listen_port_tls` と同じ | `301` リダイレクトや `Alt-Svc` ヘッダに載せる、クライアントから見える HTTPS/HTTP-3 ポートです。コンテナのポートマッピングやファイアウォール越しで公開ポートが `listen_port_tls` と異なる場合に使います。v0.13.0 で `https_redirection_port` から改名されました。 |
| `tcp_listen_backlog` | いいえ | `1024` | HTTP/1.1 および HTTP/2 リスナーの TCP listen backlog です。 |
| `max_concurrent_streams` | いいえ | `64` | 接続ごとの HTTP/2 同時ストリーム数上限です。 |
| `max_clients` | いいえ | `512` | 受け入れた HTTP/1.1 および HTTP/2 の TCP 接続数の共有上限です。TCP accept 直後にスロットが確保され、PROXY protocol の解析、TLS ハンドシェイク、接続の終了までの間保持されます。`0` を指定すると HTTP/1.1 と HTTP/2 の接続をすべて拒否します。HTTP/3 はこの上限には含まれず、[`[experimental.h3]`](#experimentalh3) の独立した上限が適用されます。 |
| `request_max_body_size` | いいえ | `268435456` (256 MiB) | HTTP/1.1、HTTP/2、HTTP/3 に共通で適用されるリクエストボディの最大サイズです。バイト単位の整数か、`"256k"`、`"10m"`、`"1g"` のような接尾辞付き文字列を指定できます。`Content-Length` が上限を超えるリクエストは upstream に接続する前に `413` で拒否され、チャンク転送やストリーミングボディの超過は転送中に検出されます。`0` または `"unlimited"` で無制限になります。 |
| `trusted_forwarded_proxies` | いいえ | なし（どのプロキシも信頼しない） | 受信した `X-Forwarded-*` / `Forwarded` ヘッダを信頼するプロキシの一覧です。CIDR ブロックと組み込みエイリアス `"cloudflare"`、`"fastly"`、`"cloudfront"` を指定できます。省略または空の場合、前段からの転送ヘッダは無視され、直接の接続元アドレスから再構築されます。詳細は [Trusted Forwarded Proxies](/docs/guide/advanced/trusted_proxies) を参照してください。 |
| `sticky_cookie_secret` | `load_balance = "sticky"` 利用時は必須 | なし | sticky セッションのクッキーを不透明な AEAD 暗号文として封印するための、パディングなし base64url でエンコードされた 32 バイトの秘密鍵です。`openssl rand -base64 32 \| tr '+/' '-_' \| tr -d '=\n'` で生成できます。秘密鍵をローテーションすると sticky セッションの割り当てはリセットされます。 |
| `redact_query_in_access_log` | いいえ | `false` | `true` にすると、アクセスログ中のクエリ文字列の値が `<redacted>` にマスクされます（パラメータのキーとパスは残ります）。トークンや PII を含む URL がログにそのまま残らないようにできます。 |
| `listen_address_v4` | いいえ | `0.0.0.0` | リスナーをバインドする IPv4 アドレスです。単一の文字列または文字列の配列を指定でき、複数のインターフェースにバインドできます。例: `['192.168.1.1', '10.0.0.1']`。複数アドレス指定時にワイルドカード `0.0.0.0` は含められません。重複アドレスは自動的に無視されます。 |
| `listen_address_v6` | いいえ | なし | リスナーをバインドする IPv6 アドレスです。単一の文字列または文字列の配列を指定できます。例: `'[::]'` や `['::1', 'fe80::1']`。省略時に `listen_ipv6 = true` であれば `[::]` にバインドします。省略時に `listen_ipv6` が `false` または未設定であれば IPv6 は無効です。複数アドレス指定時にワイルドカード `::` は含められません。重複アドレスは自動的に無視されます。 |
| `listen_ipv6` | いいえ | `false` | `true` にすると `listen_address_v6` が未指定の場合に `[::]` にバインドします。 |
| `default_app` | いいえ | なし | 平文 HTTP で `server_name` に一致しないリクエストを処理するフォールバックアプリ名です。平文 HTTP のみに適用され、不明な server name への HTTPS リクエストは無条件に拒否されます。このフォールバック経路では、送出する `Host` ヘッダはデフォルトアプリの `server_name` で強制的に上書きされ（`keep_original_host` / `set_upstream_host` より優先）、元のホスト名は `X-Forwarded-Host` / `Forwarded: host=` にのみ載ります。バックエンドはこれらの値を信頼できない参考情報として扱う必要があります。 |

## アプリケーション定義

すべてのバックエンドアプリケーションは `[apps]` 以下に定義します。

### `[apps.<app_name>]`

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `server_name` | はい | なし | このアプリが受け持つホスト名です。例: `app.example.com`。構文的に正しいホスト名である必要があります（v0.12.0 以降、起動時に検証されます）。英数字と `-` からなるドット区切りのラベルで、全体は 253 ASCII 文字までです。ワイルドカード、アンダースコア、IPv6 リテラルは拒否されます（IPv4 リテラルは使えます）。 |
| `reverse_proxy` | はい | なし | このアプリに対するルーティングルールの一覧です。 |
| `tls` | いいえ | なし | このアプリの TLS 設定です。省略すると平文 HTTP のみを提供します。 |

## TLS オプション

これらのオプションは `apps.<app_name>.tls` に書きます。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `tls_cert_path` | 静的 TLS 利用時は必須 | なし | 静的証明書を使う場合の PEM 形式サーバ証明書パスです。 |
| `tls_cert_key_path` | 静的 TLS 利用時は必須 | なし | このアプリの PEM 形式秘密鍵パスです。鍵は PKCS8 形式である必要があります。 |
| `https_redirection` | いいえ | `listen_port` と `listen_port_tls` の両方がある場合は `true` | アプリ単位の HTTP から HTTPS へのリダイレクト設定です。HTTPS のみを提供する場合は指定しないでください。 |
| `client_ca_cert_path` | いいえ | なし | mTLS のクライアント認証に使う CA 証明書パスです。このオプションを持つアプリには平文リクエストは決して転送されません。`https_redirection` が有効（デフォルト）なら `301` リダイレクトを、明示的に無効化されている場合は `421` を返します。詳細は [クライアント認証](/docs/guide/advanced/client_auth) を参照してください。 |
| `acme` | いいえ | `false` | `true` にすると `tls_cert_path` と `tls_cert_key_path` の代わりに ACME で証明書を自動取得・更新します。詳細は [ACME (Let's Encrypt) 連携](/docs/guide/advanced/acme) を参照してください。 |

## リバースプロキシ設定

各アプリには `[[apps.<app_name>.reverse_proxy]]` を 1 個以上定義できます。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `path` | いいえ | なし | `"/api"` や `"/static"` のようなパス接頭辞です。最長一致で選ばれます。 |
| `replace_path` | いいえ | 元のパスを保持 | upstream へ転送する際に置き換えるパス接頭辞です。 |
| `upstream` | はい | なし | バックエンド転送先の一覧です。 |
| `load_balance` | いいえ | `none` | バックエンド選択方式です。`none`、`round_robin`、`random`、`sticky`、`primary_backup`が使えます。`sticky` を使う場合はグローバルオプション `sticky_cookie_secret` が必須です。 |
| `upstream_options` | いいえ | なし | リクエスト転送時の挙動を制御するオプション一覧です。詳細は [Upstream Options](/docs/guide/advanced/upstream_options) を参照してください。 |
| `health_check` | いいえ | なし | アクティブヘルスチェックの設定です。デフォルトのTCPチェックには`true`を設定するか、テーブルで詳細設定できます。詳細は[アクティブヘルスチェック](/docs/guide/advanced/health_check)を参照してください。 |

## Upstream エントリ

`upstream = [...]` の各要素では次を指定できます。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `location` | はい | なし | バックエンドのホストとポートです。例: `"backend.internal:8080"` や `"www.example.com"` |
| `tls` | いいえ | `false` | `true` の場合は upstream 接続に HTTPS を使います。省略時または `false` の場合は HTTP を使います。 |

## `upstream_options` の値

各オプションの詳細な挙動は [Upstream Options](/docs/guide/advanced/upstream_options) を参照してください。ここでは利用できる値を一覧します。

| 値 | 効果 |
| --- | --- |
| `keep_original_host` | 受信した `Host` ヘッダを保持します。これがデフォルトの挙動です。 |
| `set_upstream_host` | `Host` ヘッダを upstream のホスト名に置き換えます。 |
| `upgrade_insecure_requests` | `Upgrade-Insecure-Requests: 1` がなければ追加します。 |
| `force_http11_upstream` | upstream 接続を HTTP/1.1 に固定します。 |
| `force_http2_upstream` | upstream 接続を HTTP/2 に固定します。 |
| `forwarded_header` | デフォルトの `X-Forwarded-*` ヘッダに加えて、RFC 7239 の `Forwarded` ヘッダを生成します。 |

## ヘルスチェックオプション

これらのオプションは`apps.<app_name>.reverse_proxy.health_check`に書きます。または`health_check = true`でデフォルトのTCPチェックを有効にできます。詳細は[アクティブヘルスチェック](/docs/guide/advanced/health_check)を参照してください。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `type` | いいえ | `"tcp"` | チェックタイプ: `"tcp"`または`"http"`。 |
| `interval` | いいえ | `10` | ヘルスチェックプローブ間の秒数。 |
| `timeout` | いいえ | `5` | チェック1回あたりのタイムアウト秒数。`interval`より小さい値にする必要があります。 |
| `unhealthy_threshold` | いいえ | `3` | アップストリームを異常と判定するまでの連続失敗回数。 |
| `healthy_threshold` | いいえ | `2` | アップストリームを正常と判定するまでの連続成功回数。 |
| `path` | `"http"`の場合は必須 | なし | HTTPチェックのエンドポイントパス。`/`で始まる必要があります。 |
| `expected_status` | いいえ | `200` | HTTPチェックで期待するHTTPステータスコード。 |

## Experimental 設定

`[experimental]` テーブルには、任意機能や高度な設定を書きます。

### `[experimental]`

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `ignore_sni_consistency` | いいえ | `false` | `true` にすると TLS の SNI とリクエストの `Host` ヘッダの整合性チェックを緩めます。通常は `false` のままを推奨します。この緩和はクライアント認証（`client_ca_cert_path`）を持つアプリには決して適用されず、別の server name で確立された TLS セッション経由のリクエストは常に拒否されます。 |
| `connection_handling_timeout` | いいえ | `0` | 接続全体の処理タイムアウト秒数です。`0` は無制限を意味します。 |

### `[experimental.h3]`

このテーブルを追加すると HTTP/3 を有効化します。詳細は [HTTP/3](/docs/guide/advanced/http3) を参照してください。

HTTP/3 の接続数上限はグローバルの `max_clients`（HTTP/1.1 と HTTP/2 のみ対象）とは独立しています。リクエストボディの上限はトップレベルの `request_max_body_size` で指定します。以前の `experimental.h3.request_max_body_size` キーは v0.14.0 で削除され、指定すると設定の読み込みエラーになります。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `alt_svc_max_age` | いいえ | `3600` | `Alt-Svc` の max-age 秒数です。 |
| `max_concurrent_connections` | いいえ | `512` | 設定された H3 エンドポイント/リスナーごとの HTTP/3 (QUIC) 同時接続数上限です。ハンドシェイクから接続終了までを対象とします。`0` を指定すると HTTP/3 接続をすべて拒否します。 |
| `max_concurrent_bidistream` | いいえ | `64` | 双方向 QUIC ストリーム数上限です。 |
| `max_concurrent_unistream` | いいえ | `64` | 単方向 QUIC ストリーム数上限です。 |
| `max_idle_timeout` | いいえ | `10` | QUIC のアイドルタイムアウト秒数です。`0` は無制限を意味します。 |

### `[experimental.cache]`

このテーブルを追加するとハイブリッドレスポンスキャッシュを有効化します。詳細は [キャッシュ](/docs/guide/advanced/cache) を参照してください。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `cache_dir` | いいえ | `./cache` | キャッシュディレクトリのパスです。カレントワーキングディレクトリからの相対パスです。 |
| `max_cache_entry` | いいえ | `1000` | キャッシュエントリの最大数です。 |
| `max_cache_each_size` | いいえ | `65535` | 1 レスポンスあたりのキャッシュ可能サイズ上限です。単位は bytes です。 |
| `max_cache_each_size_on_memory` | いいえ | `65535`（`max_cache_each_size` と同じ） | メモリ上に保持するキャッシュサイズ上限です。これを超えるキャッシュはファイルとして保存されます。デフォルト設定では、キャッシュ可能なオブジェクトはすべてメモリから配信され、`max_cache_each_size` をこの値より大きくした場合にファイル層が使われます。`0` にすると常にファイルキャッシュになります。最悪ケースのメモリ使用量は `max_cache_entry`×この値です。 |
| `max_cache_total_size` | いいえ | `1073741824` (1 GiB) | メモリ層とファイル層を合わせた、キャッシュが保持する総バイト数の上限です。バイト単位の整数か、`"256m"`、`"1g"` のような接尾辞付き文字列を指定できます。保存によって上限を超える場合は、最も使われていないエントリから追い出されます。`"unlimited"` で無効化できます。`0` も容量ゼロではなく無制限を意味するため、`max_cache_each_size_on_memory = 0` との混同を避けるうえでも文字列での指定を推奨します。 |

キャッシュは upstream へ転送されるリクエスト URI をキーとして保存されます。元の（クライアントから見た）ホスト・スキーム・URI によって内容が変わるバックエンド、たとえば `set_upstream_host` や `default_app` で複数の仮想ホストを 1 つの upstream に集約している場合は、キャッシュ可能なレスポンスに適切な `Vary` ヘッダを付けるか、キャッシュ不可としてマークする必要があります。

### `[experimental.acme]`

いずれかのアプリで `tls = { acme = true }` を使う場合にこのテーブルを追加します。詳細は [ACME (Let's Encrypt) 連携](/docs/guide/advanced/acme) を参照してください。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `email` | はい | なし | ACME アカウント登録に使う連絡先メールアドレスです。 |
| `dir_url` | いいえ | Let's Encrypt production directory | ACME directory URL です。 |
| `registry_path` | いいえ | `./acme_registry` | 取得した証明書やアカウント情報を保存するディレクトリです。 |

### `[experimental.tcp_recv_proxy_protocol]`

信頼できる L4 プロキシからの HAProxy PROXY protocol ヘッダを受け付ける場合にこのテーブルを追加します。詳細は [PROXY Protocol](/docs/guide/advanced/proxy_protocol) を参照してください。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `trusted_proxies` | はい | なし | 信頼するプロキシの CIDR 範囲一覧です。空は不可です。例: `["127.0.0.1/32", "::1/128"]` |
| `timeout` | いいえ | `50` ms | PROXY ヘッダ受信のタイムアウトです。単位はミリ秒です。`0` を指定すると内部的に `5s` のフォールバックタイムアウトになります。 |

## 関連ガイド

- [コマンドラインオプション](/docs/guide/cmdopt)
- [基本設定](/docs/guide/basics)
- [Upstream Options](/docs/guide/advanced/upstream_options)
- [Trusted Forwarded Proxies](/docs/guide/advanced/trusted_proxies)
- [クライアント認証](/docs/guide/advanced/client_auth)
- [HTTP/3](/docs/guide/advanced/http3)
- [キャッシュ](/docs/guide/advanced/cache)
- [ACME (Let's Encrypt) 連携](/docs/guide/advanced/acme)
- [アクティブヘルスチェック](/docs/guide/advanced/health_check)
- [PROXY Protocol](/docs/guide/advanced/proxy_protocol)
