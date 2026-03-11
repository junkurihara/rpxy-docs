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
| `https_redirection_port` | いいえ | TLS 有効時は `listen_port_tls` と同じ | `301` リダイレクトや `Alt-Svc` ヘッダに載せる外向けの HTTPS ポートです。コンテナのポートマッピングやファイアウォール越しで公開ポートが `listen_port_tls` と異なる場合に使います。 |
| `tcp_listen_backlog` | いいえ | 内部デフォルト | HTTP/1.1 および HTTP/2 リスナーの TCP listen backlog です。 |
| `max_concurrent_streams` | いいえ | 内部デフォルト | 接続ごとの HTTP/2 同時ストリーム数上限です。 |
| `max_clients` | いいえ | 内部デフォルト | HTTP/1.1、HTTP/2、HTTP/3 を合算した同時クライアント数の上限です。 |
| `listen_address_v4` | いいえ | `0.0.0.0` | リスナーをバインドする IPv4 アドレスです。 |
| `listen_address_v6` | いいえ | なし | リスナーをバインドする IPv6 アドレスです。例: `[::]`。省略時に `listen_ipv6 = true` であれば `[::]` にバインドします。省略時に `listen_ipv6` が `false` または未設定であれば IPv6 は無効です。 |
| `listen_ipv6` | いいえ | `false` | `true` にすると `listen_address_v6` が未指定の場合に `[::]` にバインドします。 |
| `default_app` | いいえ | なし | 平文 HTTP で `server_name` に一致しないリクエストを処理するフォールバックアプリ名です。HTTPS では TLS の server name を選べないため、不明なホストは拒否されます。 |

## アプリケーション定義

すべてのバックエンドアプリケーションは `[apps]` 以下に定義します。

### `[apps.<app_name>]`

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `server_name` | はい | なし | このアプリが受け持つホスト名です。例: `app.example.com` |
| `reverse_proxy` | はい | なし | このアプリに対するルーティングルールの一覧です。 |
| `tls` | いいえ | なし | このアプリの TLS 設定です。省略すると平文 HTTP のみを提供します。 |

## TLS オプション

これらのオプションは `apps.<app_name>.tls` に書きます。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `tls_cert_path` | 静的 TLS 利用時は必須 | なし | 静的証明書を使う場合の PEM 形式サーバ証明書パスです。 |
| `tls_cert_key_path` | 静的 TLS 利用時は必須 | なし | このアプリの PEM 形式秘密鍵パスです。鍵は PKCS8 形式である必要があります。 |
| `https_redirection` | いいえ | `listen_port` と `listen_port_tls` の両方がある場合は `true` | アプリ単位の HTTP から HTTPS へのリダイレクト設定です。HTTPS のみを提供する場合は指定しないでください。 |
| `client_ca_cert_path` | いいえ | なし | mTLS のクライアント認証に使う CA 証明書パスです。詳細は [クライアント認証](/docs/guide/advanced/client_auth) を参照してください。 |
| `acme` | いいえ | `false` | `true` にすると `tls_cert_path` と `tls_cert_key_path` の代わりに ACME で証明書を自動取得・更新します。詳細は [ACME (Let's Encrypt) 連携](/docs/guide/advanced/acme) を参照してください。 |

## リバースプロキシ設定

各アプリには `[[apps.<app_name>.reverse_proxy]]` を 1 個以上定義できます。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `path` | いいえ | なし | `"/api"` や `"/static"` のようなパス接頭辞です。最長一致で選ばれます。 |
| `replace_path` | いいえ | 元のパスを保持 | upstream へ転送する際に置き換えるパス接頭辞です。 |
| `upstream` | はい | なし | バックエンド転送先の一覧です。 |
| `load_balance` | いいえ | `none` | バックエンド選択方式です。`none`、`round_robin`、`random`、`sticky` が使えます。 |
| `upstream_options` | いいえ | なし | リクエスト転送時の挙動を制御するオプション一覧です。詳細は [Upstream Options](/docs/guide/advanced/upstream_options) を参照してください。 |

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

## Experimental 設定

`[experimental]` テーブルには、任意機能や高度な設定を書きます。

### `[experimental]`

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `ignore_sni_consistency` | いいえ | `false` | `true` にすると TLS バックエンドに対する SNI 整合性チェックを緩めます。通常は `false` のままを推奨します。 |
| `connection_handling_timeout` | いいえ | `0` | 接続全体の処理タイムアウト秒数です。`0` は無制限を意味します。 |

### `[experimental.h3]`

このテーブルを追加すると HTTP/3 を有効化します。詳細は [HTTP/3](/docs/guide/advanced/http3) を参照してください。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `alt_svc_max_age` | いいえ | 内部デフォルト | `Alt-Svc` の max-age 秒数です。 |
| `request_max_body_size` | いいえ | 内部デフォルト | HTTP/3 リクエストボディの最大サイズです。 |
| `max_concurrent_connections` | いいえ | 内部デフォルト | QUIC の同時接続数上限です。 |
| `max_concurrent_bidistream` | いいえ | 内部デフォルト | 双方向 QUIC ストリーム数上限です。 |
| `max_concurrent_unistream` | いいえ | 内部デフォルト | 単方向 QUIC ストリーム数上限です。 |
| `max_idle_timeout` | いいえ | 内部デフォルト | QUIC のアイドルタイムアウト秒数です。`0` は無制限を意味します。 |

### `[experimental.cache]`

このテーブルを追加するとハイブリッドレスポンスキャッシュを有効化します。詳細は [キャッシュ](/docs/guide/advanced/cache) を参照してください。

| オプション | 必須 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `cache_dir` | いいえ | `./cache` | キャッシュディレクトリのパスです。カレントワーキングディレクトリからの相対パスです。 |
| `max_cache_entry` | いいえ | `1000` | キャッシュエントリの最大数です。 |
| `max_cache_each_size` | いいえ | `65535` | 1 レスポンスあたりのキャッシュ可能サイズ上限です。単位は bytes です。 |
| `max_cache_each_size_on_memory` | いいえ | `4096` | メモリ上に保持するキャッシュサイズ上限です。これを超えるキャッシュはファイルとして保存されます。`0` にすると常にファイルキャッシュになります。 |

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
- [クライアント認証](/docs/guide/advanced/client_auth)
- [HTTP/3](/docs/guide/advanced/http3)
- [キャッシュ](/docs/guide/advanced/cache)
- [ACME (Let's Encrypt) 連携](/docs/guide/advanced/acme)
- [PROXY Protocol](/docs/guide/advanced/proxy_protocol)
