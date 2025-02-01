---
title: Using Containers
type: docs
prev: /docs/
next: /docs/container/docker
weight: 20
# sidebar:
#   open: true
---

`rpxy` docker container images are hosted both on [Docker Hub](https://hub.docker.com/r/jqtype/rpxy) and [GitHub Container Registry](https://github.com/junkurihara/rust-rpxy/pkgs/container/rust-rpxy).

This section provides information on how to use `rpxy` in a containerized environment.

## Latest and versioned builds

Latest builds are shipped from the `main` branch when the new version is released. For example, when the version `x.y.z` is released, the following images are provided.

- `latest`, `x.y.z`: Built with default features, running on Ubuntu.
- `latest-slim`, `slim`, `x.y.z-slim` : Built by `musl` with default features, running on Alpine.
- `latest-s2n`, `s2n`, `x.y.z-s2n`: Built with the `http3-s2n` feature, running on Ubuntu.

Additionally, images built with `webpki-roots` are provided in a similar manner to the above (e.g., `latest-s2n-webpki-roots` and `s2n-webpki-roots` tagged for the same image).

## Nightly builds

Nightly builds are shipped from the `develop` branch for every push.

- `nightly`: Built with default features, running on Ubuntu.
- `nightly-slim`: Built by `musl` with default features, running on Alpine.
- `nightly-s2n`: Built with the `http3-s2n` feature, running on Ubuntu.

Additionally, images built with `webpki-roots` are provided in a similar manner to the above (e.g., `nightly-s2n-webpki-roots`).

## Caveats

Due to some compile errors of `s2n-quic` subpackages with `musl`, `nightly-s2n-slim` or `latest-s2n-slim` are not yet provided.
