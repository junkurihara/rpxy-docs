---
title: rpxyについて
type: about
---

`rpxy` [ahr-pik-see] は、高速性・安全性・信頼性を重視して作られた HTTP リバースプロキシです。[Rust](https://www.rust-lang.org/) とそのエコシステムのライブラリを土台に実装されています。性能を重視する環境では、[NGINX](https://www.nginx.com/) や [Caddy](https://caddyserver.com/) の有力な代替として使えます。

シンプルな設定で扱え、メッセージ処理は高速で、安全性も既定で確保する。`rpxy` はそんな方針で設計されています。少ないリソースで多数の同時接続をさばけるのも特長です。

## クレジット

`rpxy` は、以下のプロジェクトや着想に支えられて成り立っています。

- [`hyper`](https://github.com/hyperium/hyper) と [`hyperium/h3`](https://github.com/hyperium/h3)

- [`rustls`](https://github.com/rustls/rustls)

- [`tokio`](https://github.com/tokio-rs/tokio)

- [`quinn`](https://github.com/quinn-rs/quinn)

- [`s2n-quic`](https://github.com/aws/s2n-quic)

- [`rustls-acme`](https://github.com/FlorianUekermann/rustls-acme)
