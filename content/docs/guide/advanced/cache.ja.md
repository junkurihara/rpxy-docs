---
title: キャッシュ
type: docs
prev: /docs/guide/advanced/http3
next: /docs/guide/advanced/post_quantum
weight: 4
sidebar:
  open: true
---

`rpxy`は、一時ファイルとオンメモリオブジェクトの*ハイブリッド*方式でレスポンスをキャッシュするアプローチを採用しています。
`config.toml`で`[experimental.cache]`を指定すると、一時ファイルとオンメモリオブジェクトを使用したローカルキャッシュ機能を利用できます。設定例は以下の通りです。

```toml
# これを指定するとファイルキャッシュ機能が有効になります
[experimental.cache]
cache_dir = './cache'                # オプション。デフォルトは現在の作業ディレクトリからの相対パス"./cache"
max_cache_entry = 1000               # オプション。デフォルトは1k
max_cache_each_size = 65535          # オプション。デフォルトは64k
max_cache_each_size_on_memory = 4096 # オプション。デフォルトは4k。0の場合、常にファイルキャッシュになります。
```

HTTPメッセージのコンテキストにおいて*ストア可能*なレスポンスは、サイズが`max_cache_each_size`バイト以下の場合に保存されます。さらに`max_cache_each_size_on_memory`以下の場合はオンメモリオブジェクトとして保存されます。それ以外は一時ファイルとして保存されます。`max_cache_each_size`は`max_cache_each_size_on_memory`以上である必要があります。

{{< callout type="info" >}}
`rpxy`が再起動するか設定が更新されると、キャッシュはオンメモリテーブルだけでなくファイルシステムからも完全に削除されます。
{{< /callout >}}
