---
title: キャッシュ
type: docs
prev: /docs/guide/advanced/acme
next: /docs/guide/advanced/upstream_options
weight: 4
sidebar:
  open: true
---

`rpxy`は、一時ファイルとオンメモリオブジェクトの*ハイブリッド*方式でレスポンスをキャッシュするアプローチを採用しています。
`config.toml`で`[experimental.cache]`を指定すると、一時ファイルとオンメモリオブジェクトを使用したローカルキャッシュ機能を利用できます。設定例は以下の通りです。

```toml
# これを指定するとファイルキャッシュ機能が有効になります
[experimental.cache]
cache_dir = './cache'                 # オプション。デフォルトは現在の作業ディレクトリからの相対パス"./cache"
max_cache_entry = 1000                # オプション。デフォルトは1k
max_cache_each_size = 65535           # オプション。デフォルトは64k
max_cache_each_size_on_memory = 65535 # オプション。デフォルトは64k（max_cache_each_sizeと同じ）。0の場合、常にファイルキャッシュになります。
max_cache_total_size = "1g"           # オプション。デフォルトは1 GiB。バイト単位の整数、接尾辞付き文字列（"256m"、"1g"）、または"unlimited"を指定できます。
```

HTTPメッセージのコンテキストにおいて*ストア可能*なレスポンスは、サイズが`max_cache_each_size`バイト以下の場合に保存されます。さらに`max_cache_each_size_on_memory`以下の場合はオンメモリオブジェクトとして保存されます。それ以外は一時ファイルとして保存されます。`max_cache_each_size`は`max_cache_each_size_on_memory`以上である必要があります。`max_cache_each_size_on_memory`のデフォルトは`max_cache_each_size`と同じ値のため、デフォルト設定ではキャッシュ可能なオブジェクトはすべてメモリから配信され、`max_cache_each_size`をこの値より大きくした場合にファイル層が使われます。最悪ケースのメモリ使用量は`max_cache_entry`×`max_cache_each_size_on_memory`です。

`max_cache_total_size`は、メモリ層とファイル層を合わせてキャッシュが保持する総バイト数の上限です（デフォルト1 GiB）。新しいレスポンスの保存によって上限を超える場合は、レスポンスが収まるまで最も使われていないエントリから追い出されます。`"unlimited"`で上限を無効化できます。なお`0`も容量ゼロではなく無制限を意味します。`max_cache_each_size_on_memory`では`0`が逆の意味（常にファイルキャッシュ）を持つため、文字列での指定を推奨します。

{{< callout type="warning" >}}
キャッシュはupstreamへ転送されるリクエストURIをキーとして保存されます。元の（クライアントから見た）ホスト・スキーム・URIによって内容が変わるバックエンド、たとえば`set_upstream_host`や`default_app`で複数の仮想ホストを1つのupstreamに集約している場合は、キャッシュ可能なレスポンスに適切な`Vary`ヘッダを付けるか、キャッシュ不可としてマークする必要があります。
{{< /callout >}}

{{< callout type="info" >}}
`rpxy`が再起動するか設定が更新されると、キャッシュはオンメモリテーブルだけでなくファイルシステムからも完全に削除されます。
{{< /callout >}}
