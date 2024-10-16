---
title: About
type: about
---

`rpxy` [ahr-pik-see] is designed to be a high-performance, secure, and reliable HTTP reverse-proxy. It is built on top of the [Rust](https://www.rust-lang.org/) programming language and Rust ecosystem libraries. `rpxy` is also designed to be a drop-in replacement for the popular HTTP reverse proxies such as [NGINX](https://www.nginx.com/) and [Caddy](https://caddyserver.com/), considering performance-focused environment.

`rpxy` is built under the concepts of simple configuration, blazingly fast message handling, and secure by default. It is also designed to be a high-performance HTTP reverse-proxy that can handle a large number of concurrent connections with minimal resource usage.

## Credits

`rpxy` cannot be built without the following projects and inspirations:

- [`hyper`](https://github.com/hyperium/hyper) and [`hyperium/h3`](https://github.com/hyperium/h3)

- [`rustls`](https://github.com/rustls/rustls)

- [`tokio`](https://github.com/tokio-rs/tokio)

- [`quinn`](https://github.com/quinn-rs/quinn)

- [`s2n-quic`](https://github.com/aws/s2n-quic)

- [`rustls-acme`](https://github.com/FlorianUekermann/rustls-acme)
