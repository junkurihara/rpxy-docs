---
title: ACME (Let's Encrypt) Integration
type: docs
prev: /docs/guide/advanced/cache
# next: /docs/guide/advanced/acme
weight: 2
sidebar:
  open: true
---

## Automated Certificate Issuance and Renewal via TLS-ALPN-01 ACME protocol

In addition to [use static files of certificates and private keys](/docs/guide/basics/2_tls), the automatic issuance and renewal of certificates, i.e., ACME (Automated Certificate Management Environment) standardized as [RFC8555](https://www.rfc-editor.org/rfc/rfc8555), are available in `rpxy`. To enable this feature, you need to specify the following entries for each application requiring ACME in the configuration file.

```toml:config.toml
# TLS port, which is also used for ACME challenge.
listen_port_tls = 443

# ACME enabled domain name.
# Note that acme option must be specified in the experimental section.
[apps."app_with_acme"]
server_name = 'example.org'
reverse_proxy = [{ upstream = [{ location = 'app1.local:8080' }] }]
tls = { https_redirection = true, acme = true } # do not specify tls_cert_path and/or tls_cert_key_path
```

ACME will be used to get a certificate for the `server_name` with ACME [`TLS-ALPN-01` (RFC8737)](https://www.rfc-editor.org/rfc/rfc8737) protocol. So all you need is to open
the TLS port 443 to the public. Also in this case, you don't need to specify `tls_cert_path` and `tls_cert_key_path` for the application.

For every ACME enabled domain, the following settings are referred to acquire a certificate and a private key.

```toml:config.toml
# Global ACME settings. Unless specified, ACME is disabled.
[experimental.acme]
# Email address for ACME registration.
email = "test@example.com"
# Optional: ACME directory URL. [default: "https://acme-v02.api.letsencrypt.org/directory"]
dir_url = "https://acme-staging-v02.api.letsencrypt.org/directory"
# Optional: Directory storing retrieved certificates and private keys, which is relative to the current working directory. [default: "./acme_registry"]
registry_path = "./acme_registry"
```

The above configuration is common to all ACME enabled domains. Note that the https port must be open to the public to prove the domain ownership.

{{< callout type="warning" >}}
This is a brand-new feature and maybe still unstable.
{{< /callout >}}
