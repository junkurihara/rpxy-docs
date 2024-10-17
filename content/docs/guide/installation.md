---
title: Installation
type: docs
prev: /docs/guide/
next: /docs/guide/cmdopt
weight: 1
sidebar:
  open: true
---


## Building from Source

You can build an executable binary yourself by checking out this Git repository.

```bash
# Cloning the git repository
% git clone https://github.com/junkurihara/rust-rpxy
% cd rust-rpxy

# Update submodules
% git submodule update --init

# Build
% cargo build --release
```

Then you have an executive binary `rust-rpxy/target/release/rpxy`.

{{< callout type="info" >}}
By default QUIC and HTTP/3 is enabled using `quinn`. If you want to use `s2n-quic`, build as follows. You may need several additional dependencies.

```bash
% cargo build --no-default-features --features http3-s2n --release
```
{{< /callout >}}

## Package Installation for Linux (RPM/DEB)

You can find the Jenkins CI/CD build scripts for `rpxy` in the [./.build](https://github.com/junkurihara/rust-rpxy/tree/develop/.build) directory.

Prebuilt packages for Linux RPM and DEB are available at [https://rpxy.gamerboy59.dev](https://rpxy.gamerboy59.dev), provided by [@Gamerboy59](https://github.com/Gamerboy59).

## Prebuilt Binaries

Prebuilt binaries for `amd64` (`x86_64`) and `arm64` (`aarch64`) are available at [GitHub Releases](https://github.com/junkurihara/rust-rpxy/releases/latest). The naming convention of the prebuilt binaries is as follows.

```plaintext
rpxy-${ARCH}-unknown-linux-${TARGET}-${BUILD_OPT}.tar.gz
```

| Variable | Description |
| --- | --- |
| `${ARCH}` | `x86_64` or `aarch64` |
| `${TARGET}` | `gnu` or `musl` |
| `${BUILD_OPT}` | `s2n`: Using `s2n-quic` as QUIC library. |
| | `webpki-roots`: Using [`WebPKI`](https://github.com/rustls/webpki) as the root certificate bundle for the connection with backend applications. |

{{< callout type="warning" >}}
If you are using self-signed certificates for the backend applications, you need to select non-`webpki-roots` binaries.
{{< /callout >}}


{{< callout type="info" >}}
Note that we do not have an option of installation via [`crates.io`](https://crates.io/), i.e., `cargo install`, at this point since some dependencies are not published yet. Alternatively, you can use docker image (see [Container section](/docs/container/)) as the easiest way for `amd64` environment.
{{< /callout >}}
