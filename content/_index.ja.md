---
title: rpxy
type: landing
toc: false
layout: hextra-home
---

{{< hextra/hero-badge link="https://github.com/junkurihara/rust-rpxy/releases/tag/0.11.0" >}}
  <div class="hx-w-2 hx-h-2 hx-rounded-full hx-bg-primary-400"></div>
  <span>v0.11.0</span>
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
`rpxy` [ ahr-pik-see ]
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mb-12">
{{< hextra/hero-subtitle >}}
  複数ドメインのTLS終端に対応した、&nbsp;<br class="sm:hx-block hx-hidden" />Rust製のシンプルで超高速なリバースプロキシ
{{< /hextra/hero-subtitle >}}
</div>

<div style="margin-top: 1rem; margin-bottom: 3rem;">
{{< hextra/hero-button text="はじめる" link="docs/guide/" >}}
</div>

<div style="margin-top: 1rem;"></div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="超高速"
    subtitle="Rustとそのエコシステム上に構築されたライブラリにより、`rpxy`はHTTPメッセージを超高速に処理します。実際に、`rpxy`は他の一般的なリバースプロキシと比較して速度面で優れています。"
  >}}
  {{< hextra/feature-card
    title="複数TLS終端"
    subtitle="単一の`rpxy`インスタンスで複数のドメイン名を提供できます。`rpxy`はTLS接続を処理しながら、複数のドメインを適切なバックエンドアプリケーションサーバーにルーティングします。"
  >}}
  {{< hextra/feature-card
    title="耐量子TLS"
    subtitle="ハイブリッド耐量子暗号で将来のセキュリティを確保。`rpxy`はTLS 1.3およびQUICでX25519MLKEM768をデフォルトでサポートし、パフォーマンスや特別な設定を犠牲にすることなく量子コンピューティングの脅威から保護します。"
  >}}
  {{< hextra/feature-card
    title="すぐに使えるACME"
    subtitle="`rpxy`はACMEプロトコルを標準でサポート。TLSポートを開くだけで、自動的な証明書の発行と更新によりドメインを提供できます。"
  >}}
  {{< hextra/feature-card
    title="HTTP/3対応"
    subtitle="`rpxy`はQUICベースの次世代HTTPプロトコルであるHTTP/3をサポート。最新のプロトコルでドメインを提供できます。"
  >}}
  {{< hextra/feature-card
    title="シンプルな設定"
    subtitle="`rpxy`はシンプルで簡単に設定できるよう設計されています。TOML形式のわずか数行の設定でドメインを提供できます。"
  >}}
  {{< hextra/feature-card
    title="PROXYプロトコル対応"
    subtitle="`rpxy`はAWS ELB、HAProxy、NginxなどのロードバランサーのPROXYプロトコルに対応し、プロキシチェーンを通じてオリジナルのクライアント情報を保持できます。"
  >}}
  {{< hextra/feature-card
    title="その他多数の機能..."
    icon="sparkles"
    subtitle="TLSサニタイズ、キャッシュ、ロードバランシングなど。`rpxy`はニーズに合わせて拡張性と柔軟性を備えた設計です。"
  >}}
{{< /hextra/feature-grid >}}
