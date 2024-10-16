---
title: 3. Path-based Flexible Routing
type: docs
prev: /docs/guide/basics/2_tls
next: /docs/guide/advanced
weight: 3
sidebar:
  open: true
---

`rpxy` can serves and routes incoming requests to multiple backend destination according to the path information. The routing information can be specified for each application (`server_name`) as follows.

```toml
listen_port_tls = 443

[apps.app1]
server_name = 'app1.example.com'
tls = { https_redirection = true, tls_cert_path = 'server.crt', tls_cert_key_path = 'server.key' }

[[apps.app1.reverse_proxy]]
upstream = [
  { location = 'default.backend.local' }
]

[[apps.app1.reverse_proxy]]
path = '/path'
upstream = [
  { location = 'path.backend.local' }
]

[[apps.app1.reverse_proxy]]
path = '/path/another'
replace_path = '/path'
upstream = [
  { location = 'another.backend.local' }
]
```

In the above example, a request to

```plaintext
https://app1.example.com/path/to?query=ok
```

matches the second `reverse_proxy` entry in the longest-prefix-matching manner, and will be routed to `path.backend.local` with preserving path and query information, i.e., served as

```plaintext
http://path.backend.local/path/to?query=ok
```

On the other hand, a request to

```plaintext
https://app1.example.com/path/another/xx?query=ng
```

 matching the third entry is routed with *being rewritten its path information* specified by `replace_path` option. Namely, the matched `/path/another` part is rewritten with `/path`, and it is served as

```plaintext
http://another.backend.local/path/xx?query=ng
```

Requests that doesn't match any paths will be routed by the first entry that doesn't have the `path` option, which means the *default destination*. In other words, if every `reverse_proxy` entry has an explicit `path` option, `rpxy` rejects requests that don't match any paths.

## Simple Path-based Routing Example

This path-based routing option would be enough in many cases. For example, you can serve multiple applications with one domain by specifying unique path to each application. More specifically, see the following example.

```toml
[apps.app]
server_name = 'app.example.com'
#...

[[apps.app.reverse_proxy]]
path = '/subapp1'
replace_path = '/'
upstream = [ { location = 'subapp1.local' } ]

[[apps.app.reverse_proxy]]
path = '/subapp2'
replace_path = '/'
upstream = [ { location = 'subapp2.local' } ]

[[apps.app.reverse_proxy]]
path = '/subapp3'
replace_path = '/'
upstream = [ { location = 'subapp3.local' } ]
```

This example configuration explains a very frequent situation of path-based routing. When a request to `app.example.com/subappN` routes to `sbappN.local` by replacing a path part `/subappN` to `/`.
