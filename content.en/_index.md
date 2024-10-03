---
title: rpxy
type: landing
toc: false
layout: hextra-home
---

{{< hextra/hero-badge >}}
  <div class="hx-w-2 hx-h-2 hx-rounded-full hx-bg-primary-400"></div>
  <span>Free, open source</span>
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
`rpxy` [ ahr-pik-see ]
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mb-12">
{{< hextra/hero-subtitle >}}
  A simple and ultrafast reverse-proxy serving multiple domain names&nbsp;<br class="sm:hx-block hx-hidden" />with TLS termination, written in Rust
  <!-- Fast, batteries-included Hugo theme&nbsp;<br class="sm:hx-block hx-hidden" />for creating beautiful static websites -->
{{< /hextra/hero-subtitle >}}
</div>

<div class="hx-mb-6">
{{< hextra/hero-button text="Get Started" link="docs/guide/" >}}
</div>

<div class="hx-mt-6"></div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="Blazing Fast"
    subtitle="Thanks to Rust and libraries built on top of it, `rpxy` serves HTTP messages blazing fast. In fact, `rpxy` outperforms other popular reverse-proxies in terms of the speed."
  >}}
  {{< hextra/feature-card
    title="Multiple TLS Termination"
    subtitle="You can serve multiple domain names with a single instance of `rpxy`. `rpxy` routes multiple names to appropriate backend application servers while serving TLS connections."
  >}}
  {{< hextra/feature-card
    title="Simple Configuration"
    subtitle="`rpxy` is designed to be simple and easy to configure. You can serve your domain names with just a few lines of configuration in TOML format."
  >}}
  {{< hextra/feature-card
    title="ACME Out of the Box"
    subtitle="`rpxy` supports ACME protocol out of the box. You can serve your domain names with automatic certificate issuance and renewal only by opening a port for TLS."
  >}}
  {{< hextra/feature-card
    title="HTTP/3 Support"
    subtitle="`rpxy` supports HTTP/3, the next generation of the HTTP protocol based on QUIC. You can serve your domain names with the latest protocol."
  >}}
  {{< hextra/feature-card
    title="And Much More..."
    icon="sparkles"
    subtitle="TLS sanitization, caching, load balancing, and more. `rpxy` is designed to be extensible and flexible for your needs."
  >}}
{{< /hextra/feature-grid >}}

<!--
## rpxy: A simple and ultrafast reverse-proxy serving multiple domain names with TLS termination, written in Rust -->

<!-- `rpxy` [ahr-pik-see] is an implementation of simple and lightweight reverse-proxy with some additional features. -->
