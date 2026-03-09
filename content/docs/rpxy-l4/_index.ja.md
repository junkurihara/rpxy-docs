---
title: rpxy-l4
type: docs
prev: /docs/
weight: 35
---

[`rpxy-l4`](https://github.com/junkurihara/rust-rpxy-l4) は `rpxy` の姉妹プロジェクトです。`rpxy` が Layer 7 の HTTP(S) リバースプロキシに注力しているのに対し、`rpxy-l4` は TCP/UDP を対象とした Layer 4 プロキシを扱います。

{{< callout type="warning" >}}
`rpxy-l4` は、すでに一部の実環境で動作している一方で、作者の研究的関心から発展している実験的プロジェクトでもあります。このページは概要紹介に留め、詳細なセットアップや運用方法は上流リポジトリを正としてください。
{{< /callout >}}

## `rpxy-l4` が向いている用途

`rpxy-l4` は、HTTP より手前でトラフィックを扱いたい場合や、そもそも HTTP ではない通信をさばきたい場合に向いています。

代表的な用途は次の通りです。

- TCP / UDP の転送
- 同一ポート上でのプロトコル多重化
- TLS / QUIC に対する SNI / ALPN ベースの振り分け
- HAProxy PROXY protocol によるクライアント IP の保持

## どちらを使うべきか

| 必要なもの | 使うもの |
| --- | --- |
| HTTP/1.1、HTTP/2、HTTP/3 のリバースプロキシ、TLS 終端、ACME、パスベースルーティング | `rpxy` |
| TCP/UDP プロキシ、プロトコル単位の振り分け、`rpxy` の前段に置く L4 プロキシ | `rpxy-l4` |
| HTTP ルーティングに加えて、クライアント IP 保持や L4 レベルのトラフィック制御も必要 | `rpxy-l4` + `rpxy` |

## `rpxy-l4` と `rpxy` を組み合わせる例

よくある構成の 1 つは次の通りです。

```plaintext
Client --> rpxy-l4 --> rpxy --> Backend App
```

この構成では、`rpxy-l4` が TCP 転送や PROXY protocol のような Layer 4 の役割を担い、`rpxy` が HTTP ルーティング、TLS 終端、ACME、キャッシュなど Layer 7 の役割を担当します。

## 詳細情報の参照先

現時点での [`rpxy-l4`](https://github.com/junkurihara/rust-rpxy-l4) の主な参照先は上流リポジトリです。

- GitHub: [junkurihara/rust-rpxy-l4](https://github.com/junkurihara/rust-rpxy-l4)
- このワークスペース内のローカル checkout: `../rust-rpxy-l4`

`rpxy` との接続点として特に関連が深い内容は、[PROXYプロトコル](/docs/guide/advanced/proxy_protocol) を参照してください。
