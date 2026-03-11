---
title: rpxy
type: landing
toc: false
layout: hextra-home
---

{{< hextra/hero-badge link="https://github.com/junkurihara/rust-rpxy/releases/tag/0.11.1" >}}
  <div class="hx-w-2 hx-h-2 hx-rounded-full hx-bg-primary-400"></div>
  <span>v0.11.1</span>
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
`rpxy` [ ahr-pik-see ]
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mb-12">
{{< hextra/hero-subtitle >}}
  Rust で書かれた、&nbsp;<br class="sm:hx-block hx-hidden" />複数ドメイン・TLS 終端対応のシンプルかつ超高速なリバースプロキシ
{{< /hextra/hero-subtitle >}}
</div>

<div style="margin-top: 1rem; margin-bottom: 3rem;">
{{< hextra/hero-button text="はじめる" link="docs/guide/" >}}
</div>

<div style="margin-top: 1rem;"></div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="超高速"
    subtitle="Rust の性能と周辺ライブラリの強みを活かし、`rpxy` は HTTP トラフィックを高速にさばきます。主要なリバースプロキシと比べても、高い性能を発揮します。"
  >}}
  {{< hextra/feature-card
    title="複数TLS終端"
    subtitle="1 つの `rpxy` インスタンスで複数ドメインをまとめて扱えます。TLS を終端しながら、各ドメインを適切なバックエンドへ振り分けます。"
  >}}
  {{< hextra/feature-card
    title="耐量子TLS"
    subtitle="ハイブリッド耐量子暗号により、将来の量子計算リスクにも備えられます。TLS 1.3 と QUIC では X25519MLKEM768 を標準で利用できます。"
  >}}
  {{< hextra/feature-card
    title="すぐに使えるACME"
    subtitle="ACME に標準対応。TLS 用のポートを開けるだけで、証明書の取得と更新を自動化できます。"
  >}}
  {{< hextra/feature-card
    title="HTTP/3対応"
    subtitle="QUIC ベースの HTTP/3 に対応しています。最新プロトコルをそのまま使って配信できます。"
  >}}
  {{< hextra/feature-card
    title="シンプルな設定"
    subtitle="設定は TOML で数行書くだけ。複雑な前準備なしで使い始められます。"
  >}}
  {{< hextra/feature-card
    title="PROXYプロトコル対応"
    subtitle="AWS ELB、HAProxy、Nginx などの背後でも動作し、PROXY プロトコル経由で元のクライアント情報を引き継げます。"
  >}}
  {{< hextra/feature-card
    title="その他多数の機能..."
    icon="sparkles"
    subtitle="TLS サニタイズ、キャッシュ、ロードバランシングなどにも対応。用途に応じて柔軟に広げられます。"
  >}}
{{< /hextra/feature-grid >}}
