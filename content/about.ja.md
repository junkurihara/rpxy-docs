---
title: rpxyについて
type: about
---

`rpxy` [ahr-pik-see]は、高性能で安全かつ信頼性の高いHTTPリバースプロキシとして設計されています。[Rust](https://www.rust-lang.org/)プログラミング言語とRustエコシステムのライブラリを基盤に構築されています。`rpxy`は、パフォーマンス重視の環境において、[NGINX](https://www.nginx.com/)や[Caddy](https://caddyserver.com/)などの一般的なHTTPリバースプロキシの代替として使用できるよう設計されています。

`rpxy`は、シンプルな設定、超高速なメッセージ処理、デフォルトで安全という理念のもとに構築されています。また、最小限のリソース使用量で大量の同時接続を処理できる高性能HTTPリバースプロキシとして設計されています。

## クレジット

`rpxy`は以下のプロジェクトとインスピレーションなしには構築できませんでした:

- [`hyper`](https://github.com/hyperium/hyper) and [`hyperium/h3`](https://github.com/hyperium/h3)

- [`rustls`](https://github.com/rustls/rustls)

- [`tokio`](https://github.com/tokio-rs/tokio)

- [`quinn`](https://github.com/quinn-rs/quinn)

- [`s2n-quic`](https://github.com/aws/s2n-quic)

- [`rustls-acme`](https://github.com/FlorianUekermann/rustls-acme)
