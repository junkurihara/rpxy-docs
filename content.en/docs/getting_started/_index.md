---
weight: 1
bookFlatSection: true
bookCollapseSection: true
title: "Getting Started"
---

# Getting Started

## Building/Installation of `rpxy`

----

### Building from Source

You can build an executable binary yourself by checking out this Git repository.

```bash
# Cloning the git repository
% git clone https://github.com/junkurihara/rust-rpxy
% cd rust-rpxy

# Update submodules
% git submodule update --init

# Build (default: QUIC and HTTP/3 is enabled using `quinn`)
% cargo build --release

# If you want to use `s2n-quic`, build as follows. You may need several additional dependencies.
% cargo build --no-default-features --features http3-s2n --release
```

Then you have an executive binary `rust-rpxy/target/release/rpxy`.

{{< hint info >}}
Note that we do not have an option of installation via [`crates.io`](https://crates.io/), i.e., `cargo install`, at this point since some dependencies are not published yet. Alternatively, you can use docker image (see below) as the easiest way for `amd64` environment.
{{< /hint >}}

----

### Package Installation for Linux (RPM/DEB)

You can find the Jenkins CI/CD build scripts for `rpxy` in the [./.build](https://github.com/junkurihara/rust-rpxy/tree/develop/.build) directory.

Prebuilt packages for Linux RPM and DEB are available at [https://rpxy.gamerboy59.dev](https://rpxy.gamerboy59.dev), provided by [@Gamerboy59](https://github.com/Gamerboy59).

----

{{< next-button relref="cmd_options" >}}Next: Command Line Options{{< /next-button >}}
