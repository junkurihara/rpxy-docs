---
title: Basic Setups
type: docs
prev: /docs/guide/cmdopt
next: /docs/guide/basics/1_cleartext
weight: 3
# sidebar:
#   open: true
---

Here we explain some typical setups for `rpxy`. We will start with the most basic setups.

{{< cards >}} {{< card link="1_cleartext" title="1. Cleartext HTTP" icon="map" >}}
{{< card link="2_tls" title="2. TLS Termination" icon="key" >}}
{{< card link="3_routing" title="3. Routing" icon="globe" >}}
{{< /cards >}}

Wwe frequently add new options for `rpxy` configuration. Then, we first add new option entries in [Configuration Options](/docs/guide/configuration) section and the [`config-example.toml`](https://github.com/junkurihara/rust-rpxy/blob/develop/config-example.toml) file. So please refer to them for up-to-date options.
