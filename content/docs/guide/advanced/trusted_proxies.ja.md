---
title: 信頼する転送プロキシ
type: docs
prev: /docs/guide/advanced/upstream_options
next: /docs/guide/advanced/post_quantum
weight: 6
sidebar:
  open: true
---

## 概要

`rpxy`が別のロードバランサー、リバースプロキシ、CDNの背後で動作する場合、元のクライアント情報（IPアドレスやプロトコルスキーム）は`X-Forwarded-For`や`Forwarded`などの転送ヘッダーで届きます。これらのヘッダーはクライアントが簡単に偽装できるため、実際に管理・信頼しているプロキシから届いた場合にのみ信頼すべきものです。

v0.12.0以降、**デフォルトではどのプロキシも信頼されません**。`rpxy`は受信した転送ヘッダーを無視し、`X-Forwarded-For`、`Forwarded`および関連ヘッダーを直接の接続元アドレスのみから再構築します。信頼できる前段プロキシがある構成では、グローバルオプション`trusted_forwarded_proxies`でチェーンを保持できます。

## 設定

```toml
# 前段プロキシのCIDRブロック
trusted_forwarded_proxies = ["10.0.0.0/8", "192.168.0.0/16"]

# 組み込みのプロバイダーエイリアスはCIDRと混在できます
trusted_forwarded_proxies = ["cloudflare", "10.0.0.0/8"]

# すべてのIPv4プロキシを信頼（非推奨）
trusted_forwarded_proxies = "0.0.0.0/0"
# デュアルスタックですべて信頼する場合は"0.0.0.0/0"と"::/0"の両方を指定します。
```

受信接続の直接の接続元が信頼された範囲内にある場合、`rpxy`は信頼されたプロキシ経由で得た転送情報を保持・正規化し、正規化されたチェーンから送出する`X-Forwarded-For`および関連ヘッダーを再構築します。受信した転送情報が不正または矛盾している場合は、直接の接続元のみの情報に安全にフォールバックします。

### 組み込みのプロバイダーエイリアス

エイリアス`"cloudflare"`、`"fastly"`、`"cloudfront"`は、各プロバイダーが公開しているIPレンジに展開されます。レンジは設定読み込み時に解決される組み込みの静的スナップショットであり、`rpxy`が実行時にプロバイダーのIPリストを取得することはありません。そのため、起動が外部ネットワークの到達性に依存することもありません。スナップショットは[rust-rpxyリポジトリ](https://github.com/junkurihara/rust-rpxy/tree/main/rpxy-trusted-proxies)の`rpxy-trusted-proxies`アップデーターで明示的に更新できます。

## 運用上の要件

{{< callout type="warning" >}}
`trusted_forwarded_proxies`に列挙するプロキシは、クライアントが指定した値を追記するのではなく、受信した`X-Forwarded-Proto`（および関連ヘッダー）を**上書きまたは正規化**しなければなりません。たとえばNginxでは`proxy_set_header X-Forwarded-Proto $scheme;`とします。そうでない場合、信頼されたプロキシのさらに前段にいる攻撃者がスキームやチェーンを偽装できてしまいます。AWS ALBとCloudFrontはデフォルトでこの要件を満たします。
{{< /callout >}}

## Stickyセッションとの関係

Stickyセッション（`load_balance = "sticky"`）のクッキーの`Secure`属性は、クライアントから見たリクエストのスキームがHTTPSの場合に付与されます。`rpxy`自身がTLSを終端する場合は自動的に付与されます。`rpxy`が外部のTLS終端（ALB、CloudFront、Nginx、HAProxyなど）の背後で動作する場合、`rpxy`は信頼されたピアからの`X-Forwarded-Proto: https`（または`Forwarded: proto=https`）のみを受け入れるため、`Secure`を付与するにはその終端のアドレスを`trusted_forwarded_proxies`に列挙する必要があります。

## PROXYプロトコルとの関係

`trusted_forwarded_proxies`は**HTTPレベル**の転送ヘッダーの信頼を制御します。L4プロキシから**TCPレベル**で元のクライアントアドレスを復元するには、[PROXYプロトコル](/docs/guide/advanced/proxy_protocol)サポート（`[experimental.tcp_recv_proxy_protocol]`）を使用してください。こちらは独立した`trusted_proxies`リストを持ちます。
