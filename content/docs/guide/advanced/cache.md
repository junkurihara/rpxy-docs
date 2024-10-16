---
title: Caching
type: docs
prev: /docs/guide/advanced/http3
# next: /docs/guide/advanced/acme
weight: 4
sidebar:
  open: true
---

`rpxy` takes an approach to cache responses in a *hybrid* way of temporary files and on-memory objects.
If `[experimental.cache]` is specified in `config.toml`, you can leverage the local caching feature using temporary files and on-memory objects. An example configuration is as follows.

```toml
# If this specified, file cache feature is enabled
[experimental.cache]
cache_dir = './cache'                # optional. default is "./cache" relative to the current working directory
max_cache_entry = 1000               # optional. default is 1k
max_cache_each_size = 65535          # optional. default is 64k
max_cache_each_size_on_memory = 4096 # optional. default is 4k if 0, it is always file cache.
```

A *storable* (in the context of an HTTP message) response is stored if its size is less than or equal to `max_cache_each_size` in bytes. If it is also less than or equal to `max_cache_each_size_on_memory`, it is stored as an on-memory object. Otherwise, it is stored as a temporary file. Note that `max_cache_each_size` must be larger or equal to `max_cache_each_size_on_memory`.

{{< callout type="info" >}}
Once `rpxy` restarts or the config is updated, the cache is totally eliminated not only from the on-memory table but also from the file system.
{{< /callout >}}
