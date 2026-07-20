---
title: Caching
type: docs
prev: /docs/guide/advanced/acme
next: /docs/guide/advanced/upstream_options
weight: 4
sidebar:
  open: true
---

`rpxy` takes an approach to cache responses in a *hybrid* way of temporary files and on-memory objects.
If `[experimental.cache]` is specified in `config.toml`, you can leverage the local caching feature using temporary files and on-memory objects. An example configuration is as follows.

```toml
# If this specified, file cache feature is enabled
[experimental.cache]
cache_dir = './cache'                 # optional. default is "./cache" relative to the current working directory
max_cache_entry = 1000                # optional. default is 1k
max_cache_each_size = 65535           # optional. default is 64k
max_cache_each_size_on_memory = 65535 # optional. default is 64k, same as max_cache_each_size. if 0, it is always file cache.
max_cache_total_size = "1g"           # optional. default is 1 GiB. accepts an integer (bytes) or a suffixed string ("256m", "1g"), or "unlimited".
```

A *storable* (in the context of an HTTP message) response is stored if its size is less than or equal to `max_cache_each_size` in bytes. If it is also less than or equal to `max_cache_each_size_on_memory`, it is stored as an on-memory object. Otherwise, it is stored as a temporary file. Note that `max_cache_each_size` must be larger or equal to `max_cache_each_size_on_memory`. Since `max_cache_each_size_on_memory` defaults to the same value as `max_cache_each_size`, every cacheable object is served from memory by default; the file tier engages when you raise `max_cache_each_size` beyond it. The worst-case on-memory footprint is `max_cache_entry` × `max_cache_each_size_on_memory`.

`max_cache_total_size` caps the total bytes retained by the cache across the on-memory and file tiers (default 1 GiB). When storing a new response would exceed the ceiling, the least recently used entries are evicted until the response fits. Set `"unlimited"` to disable the ceiling. Note that `0` also means unlimited, not zero capacity; the string form is recommended since `0` has the opposite meaning ("always file cache") for `max_cache_each_size_on_memory`.

{{< callout type="warning" >}}
Cached responses are keyed on the request URI forwarded upstream. A backend that varies content by the original client-facing host, scheme, or URI — for example, multiple virtual hosts flattened onto one upstream via `set_upstream_host` or `default_app` — must emit an appropriate `Vary` header on cacheable responses or mark them non-cacheable.
{{< /callout >}}

{{< callout type="info" >}}
Once `rpxy` restarts or the config is updated, the cache is totally eliminated not only from the on-memory table but also from the file system.
{{< /callout >}}
