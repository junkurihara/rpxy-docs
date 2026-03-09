---
title: 3. パスベースの柔軟なルーティング
type: docs
prev: /docs/guide/basics/2_tls
next: /docs/guide/advanced
weight: 3
sidebar:
  open: true
---

`rpxy`は、パス情報に基づいて受信リクエストを複数のバックエンド先にルーティングできます。ルーティング情報は各アプリケーション（`server_name`）ごとに以下のように指定できます。

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

上記の例では、以下へのリクエスト

```plaintext
https://app1.example.com/path/to?query=ok
```

は最長プレフィックスマッチングにより2番目の`reverse_proxy`エントリに一致し、パスとクエリ情報を保持したまま`path.backend.local`にルーティングされます。つまり、以下のように処理されます。

```plaintext
http://path.backend.local/path/to?query=ok
```

一方、以下へのリクエスト

```plaintext
https://app1.example.com/path/another/xx?query=ng
```

は3番目のエントリに一致し、`replace_path`オプションで指定された*パス情報の書き換え*を伴ってルーティングされます。つまり、一致した`/path/another`部分が`/path`に書き換えられ、以下のように処理されます。

```plaintext
http://another.backend.local/path/xx?query=ng
```

いずれのパスにも一致しないリクエストは、`path`オプションを持たない最初のエントリによってルーティングされ、これが*デフォルトの転送先*となります。言い換えれば、すべての`reverse_proxy`エントリに明示的な`path`オプションがある場合、`rpxy`はいずれのパスにも一致しないリクエストを拒否します。

## シンプルなパスベースルーティングの例

このパスベースルーティングオプションは多くのケースで十分です。例えば、各アプリケーションに固有のパスを指定することで、1つのドメインで複数のアプリケーションを提供できます。具体的には以下の例を参照してください。

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

この設定例は、パスベースルーティングの非常によくある状況を示しています。`app.example.com/subappN`へのリクエストがパス部分`/subappN`を`/`に置換して`subappN.local`にルーティングされます。
