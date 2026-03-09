---
title: PROXYプロトコル
type: docs
prev: /docs/guide/advanced/post_quantum
weight: 6
sidebar:
  open: true
---

## PROXYプロトコルとは?

リバースプロキシやロードバランサーがクライアントとバックエンドサーバーの間に存在する場合、バックエンドサーバーにはプロキシのIPアドレスがソースとして見え、元のクライアントのアドレスは見えません。[PROXYプロトコル](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)は、**TCPレイヤー**でこの問題を解決します。上流のプロキシが各TCP接続の先頭に、元のクライアントのIPアドレスとポートを含む小さなヘッダーを付加します。

`X-Forwarded-For`や`Forwarded`ヘッダーなどのHTTPレベルのソリューションとは異なり、PROXYプロトコルはTCPベースのあらゆるプロトコルに対して透過的に動作し、アプリケーションレベルのクライアントによるなりすましができません。

## rpxyでの動作

`rpxy`は**受信**TCP接続でのPROXYプロトコルv1/v2ヘッダーの受信をサポートしています。これにより、`rpxy`はPROXYプロトコルに対応したL4ロードバランサーやプロキシ（AWS ELB、HAProxy、Nginx、[rpxy-l4](https://github.com/junkurihara/rpxy-l4)など）の背後に配置でき、`rpxy`はPROXYヘッダーから元のクライアントのソースIPとポートを復元します。

一般的なデプロイ構成は以下の通りです:

```plaintext
Client --> L4 Load Balancer (PROXY protocol) --> rpxy --> Backend App
```

PROXYプロトコルがない場合、`rpxy`にはロードバランサーのIPのみがクライアントアドレスとして見えます。PROXYプロトコルを有効にすると、`rpxy`は付加されたヘッダーを解析し、ログと転送に実際のクライアントIPを使用します。

## 設定

PROXYプロトコルサポートはデフォルトビルドに含まれています。有効にするには、設定ファイルに`[experimental.tcp_recv_proxy_protocol]`セクションを追加します:

```toml
[experimental.tcp_recv_proxy_protocol]
trusted_proxies = ["127.0.0.1/32", "10.0.0.0/8"]  # 必須、空でないCIDRリスト（IPv4および/またはIPv6）
timeout = 50  # オプション、ミリ秒。デフォルト: 50ms。0 = 5sにフォールバック。
```

| パラメータ | 説明 |
| --- | --- |
| `trusted_proxies` | **必須**。信頼されたプロキシソースを指定するCIDR範囲の空でないリスト。例: `["127.0.0.1/32", "::1/128"]` |
| `timeout` | オプション。PROXYヘッダー受信のタイムアウト（ミリ秒）。デフォルトは50ms。0に設定すると5sにフォールバック（非推奨）。 |

有効にすると、`rpxy`は信頼されたプロキシからのすべての受信TCP接続にPROXYプロトコルヘッダーが含まれることを期待し、そこから元のクライアントアドレス情報を抽出します。

{{< callout type="error" >}}
**セキュリティ警告**: PROXYヘッダーは認証されません。設定時は、**すべてのTCP接続がリストされた信頼されたプロキシから送信される必要があります**。クライアントアドレスのなりすましを防ぐため、ファイアウォールルールでアクセスを制限してください。
{{< /callout >}}

## 制限事項

{{< callout type="warning" >}}
QUICの基盤となるUDPはコネクションレスであるため、PROXYプロトコルで**HTTP/3 (QUIC)はサポートされていません**。PROXYプロトコルサポートはTCP上のHTTP/1.1およびHTTP/2にのみ適用されます。
{{< /callout >}}

## ユースケース

- **AWS Elastic Load Balancer (ELB)**の背後での`rpxy`の運用
- PROXYプロトコルが有効な**HAProxy**の背後での`rpxy`の運用
- TCPプロキシとしてPROXYプロトコル付きで設定された**Nginx**の背後での`rpxy`の運用
- レイヤー4ロードバランシングのための**rpxy-l4**との連携

`rpxy-l4` と `rpxy` の関係を手短に確認したい場合は、[rpxy-l4](/docs/rpxy-l4) を参照してください。
