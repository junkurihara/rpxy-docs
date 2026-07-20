---
title: アクティブヘルスチェック
type: docs
prev: /docs/guide/advanced/proxy_protocol
weight: 9
sidebar:
  open: true
---

## 概要

`rpxy`が複数のアップストリームバックエンドにリクエストを転送する場合、ダウンしているサーバーや異常なサーバーにトラフィックを送らないことが重要です。**アクティブヘルスチェック**機能は、各アップストリームを定期的にプローブし、異常なものをロードバランシングプールから自動的に除外します。

主な動作:

- **TCP**または**HTTP**のプローブタイプ
- `[[apps.<name>.reverse_proxy]]`ブロックごとに設定可能
- 設定された回数の連続失敗でアップストリームが異常と判定され、連続成功で正常に復帰
- 起動時に即座にプローブを実行し、その後は固定間隔でプローブ
- **すべての**アップストリームが異常になった場合、`rpxy`は警告をログに出力し、ベストエフォートでルーティングを継続

{{< callout type="info" >}}
アクティブヘルスチェックにはビルド時に`health-check` Cargoフィーチャーが有効である必要があります。このフィーチャーが有効なビルド済みバイナリとDockerイメージは公式リリースから入手できます。
{{< /callout >}}

## 設定

### 最も簡単な形式 — デフォルトでのTCPチェック

`health_check = true`を設定すると、すべてのデフォルトパラメータでTCP接続チェックが有効になります:

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "backend1:8080" },
  { location = "backend2:8080" },
]
health_check = true
```

### TCPの全設定

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "backend1:8080" },
  { location = "backend2:8080" },
]

[apps.app1.reverse_proxy.health_check]
type = "tcp"
interval = 10
timeout = 5
unhealthy_threshold = 3
healthy_threshold = 2
```

### HTTPヘルスチェック

HTTPヘルスチェックでは、`rpxy`が指定されたパスにHTTPリクエストを送信し、レスポンスのステータスコードを確認します:

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "backend1:8080" },
  { location = "backend2:8080" },
]

[apps.app1.reverse_proxy.health_check]
type = "http"
path = "/healthz"
expected_status = 200
interval = 10
timeout = 5
unhealthy_threshold = 3
healthy_threshold = 2
```

## オプション

| オプション | デフォルト | 説明 |
| --- | --- | --- |
| `type` | `"tcp"` | チェックタイプ: `"tcp"`または`"http"`。 |
| `interval` | `10` | ヘルスチェックプローブ間の秒数。 |
| `timeout` | `5` | チェック1回あたりのタイムアウト秒数。`interval`より小さい値にする必要があります。 |
| `unhealthy_threshold` | `3` | アップストリームを異常と判定するまでの連続失敗回数。 |
| `healthy_threshold` | `2` | アップストリームを正常と判定するまでの連続成功回数。 |
| `path` | — | HTTPチェックのエンドポイントパス。`type = "http"`の場合は**必須**。`/`で始まる必要があります。 |
| `expected_status` | `200` | HTTPチェックで期待するHTTPステータスコード。 |

## ロードバランシングとの連携

ヘルスチェック機能は`load_balance`オプションと連携して動作します:

| `load_balance` | ヘルスチェック時の動作 |
| --- | --- |
| `"none"` | ヘルスチェックなしの場合、常に最初のアップストリームを選択。ヘルスチェックありの場合、**最初の正常な**アップストリームを選択。 |
| `"round_robin"` | ローテーション中に異常なアップストリームをスキップ。 |
| `"random"` | 正常なアップストリームからのみランダムに選択。 |
| `"sticky"` | スティッキーターゲットが異常な場合、別の正常なアップストリームにフォールバック。 |
| `"primary_backup"` | 常に最初の正常なアップストリームにルーティング。`health_check`の有効化が**必須**。 |

{{< callout type="warning" >}}
`"primary_backup"`ロードバランスモードには`health_check`の設定が必要です。`health_check`設定なしで`load_balance = "primary_backup"`を設定すると、`rpxy`は起動時にエラーを返します。
{{< /callout >}}

## 例: プライマリ/バックアップ構成とHTTPヘルスチェック

プライマリがダウンした場合にのみバックアップがトラフィックを受け取る一般的なパターンです:

```toml
[[apps.app1.reverse_proxy]]
upstream = [
  { location = "primary.internal:8080" },
  { location = "backup.internal:8080" },
]
load_balance = "primary_backup"

[apps.app1.reverse_proxy.health_check]
type = "http"
path = "/healthz"
interval = 5
unhealthy_threshold = 2
healthy_threshold = 1
```

この設定では、`rpxy`は`primary.internal:8080`が正常な限り、常にそこにトラフィックを送信します。プライマリが2回連続でヘルスチェックに失敗すると、トラフィックは`backup.internal:8080`に切り替わります。プライマリが1回ヘルスチェックに合格すると、トラフィックはプライマリに戻ります。
