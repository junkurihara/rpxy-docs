---
title: Post-Quantum Cryptography
type: docs
prev: /docs/guide/advanced/cache
# next: /docs/guide/advanced/cache
weight: 5
sidebar:
  open: true
---

## Overview

`rpxy` provides cutting-edge security by supporting post-quantum cryptography, protecting your applications against future quantum computing threats. This feature is enabled by default and requires no special configuration.

## What is Post-Quantum Cryptography?

Post-quantum cryptography refers to cryptographic algorithms that are secure against attacks by quantum computers. As quantum computing technology advances, traditional cryptographic methods like RSA and ECC may become vulnerable. Post-quantum algorithms are designed to remain secure even when powerful quantum computers become available.

## rpxy's Implementation

### Technical Foundation

`rpxy` enables post-quantum cryptography through the [`rustls-post-quantum`](https://github.com/rustls/rustls/tree/main/rustls-post-quantum) library, which provides post-quantum key exchange algorithms for rustls. This integration allows `rpxy` to offer cutting-edge cryptographic security with minimal configuration.

### Hybrid Key Exchange

`rpxy` implements **hybrid post-quantum key exchange** that combines:

- **Classical cryptography** (X25519) - Proven security for current threats
- **Post-quantum cryptography** (ML-KEM 768) - Protection against quantum attacks

This hybrid approach provides:

- **Backward compatibility** with existing systems
- **Future-proof security** against quantum threats
- **Performance optimization** without sacrificing security

### Supported Algorithms

{{< callout type="info" >}}
**X25519MLKEM768**: This is the hybrid key exchange algorithm supported by `rpxy`, combining X25519 elliptic curve cryptography with ML-KEM 768 (formerly Kyber768) post-quantum cryptography.
{{< /callout >}}

### Protocol Coverage

Post-quantum cryptography is supported across all secure protocols:

- **TLS 1.3** - All HTTPS connections use post-quantum key exchange
- **QUIC and HTTP/3** - Post-quantum security for next-generation protocols
- **Automatic negotiation** - Clients and servers negotiate the best available algorithm

## Configuration

### Default Behavior

Post-quantum cryptography is **enabled by default** in `rpxy`. No configuration changes are required:

```toml
# Post-quantum TLS is automatically enabled for all applications
[apps."secure_app"]
server_name = 'example.com'
reverse_proxy = [{ upstream = [{ location = 'backend:8080' }] }]
tls = { https_redirection = true }
```

### Verification

You can verify post-quantum key exchange is active by:

1. **Checking TLS handshake logs** - Look for `MLKEM768` or something like that in debug output
2. **Using OpenSSL s_client** - Examine cipher suite negotiation
3. **Network analysis tools** - Inspect TLS handshake packets

## Security Benefits

### Current Protection

- **Traditional threats** - RSA, ECC still secure against classical computers
- **Side-channel attacks** - Rust's memory safety provides additional protection
- **Perfect forward secrecy** - Session keys remain secure even if long-term keys are compromised

### Future Protection

- **Quantum resistance** - Algorithms designed to withstand quantum attacks
- **Long-term security** - Data encrypted today remains secure in the quantum era
- **Cryptographic agility** - Easy transition to new algorithms as standards evolve

## Performance Impact

{{< callout type="info" >}}
**Minimal Overhead**: The hybrid approach ensures post-quantum security while maintaining excellent performance. Key exchange operations are optimized and cached where possible.
{{< /callout >}}

## Industry Standards

`rpxy`'s post-quantum implementation follows:

- **IETF drafts** - Implements [draft-kwiatkowski-tls-ecdhe-mlkem-03](https://www.ietf.org/archive/id/draft-kwiatkowski-tls-ecdhe-mlkem-03.html) and [draft-ietf-tls-hybrid-design](https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/)
- **NIST standards** - Uses standardized ML-KEM (FIPS 203) cryptographic primitives
- **Industry adoption** - Compatible with other implementations following the same IETF drafts

{{< callout type="info" >}}
**Standard Status**: The X25519MLKEM768 algorithm is based on IETF draft specifications ([draft-kwiatkowski-tls-ecdhe-mlkem-03](https://www.ietf.org/archive/id/draft-kwiatkowski-tls-ecdhe-mlkem-03.html) and [draft-ietf-tls-hybrid-design](https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/)) while using the standardized ML-KEM (FIPS 203) post-quantum cryptographic primitives.
{{< /callout >}}

## Compatibility

### Client Support

Modern browsers and HTTP clients increasingly support post-quantum key exchange:

- **Chrome/Chromium** - Experimental support available
- **Firefox** - Support in development
- **OpenSSL** - Recent versions support hybrid algorithms

### Fallback Behavior

If clients don't support post-quantum algorithms, `rpxy` automatically falls back to classical cryptography, ensuring compatibility while providing enhanced security when possible.
