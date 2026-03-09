---
title: コマンドラインオプション
type: docs
prev: /docs/guide/installation
next: /docs/guide/basics
weight: 2
sidebar:
  open: true
---

`rpxy`は常にTOML形式の設定ファイルを必要とします（例: [GitHubリポジトリの`config.toml`](https://github.com/junkurihara/rust-rpxy/blob/main/config-example.toml)）。

{{< callout type="info" >}}
代表的な設定については[基本設定](/docs/guide/basics)と[高度な使い方](/docs/guide/advanced)セクションで紹介します。
詳細な設定オプションは[設定オプション](/docs/configuration)セクションで説明しています。
{{< /callout >}}

以下のように設定ファイルを指定して`rpxy`を実行できます。

```bash
% ./path/to/rpxy --config config.toml
```

バージョン0.10.0以降、`rpxy`は設定ファイルの変更をリアルタイムで自動監視し、プロセスを再起動せずに即座に反映します。

ヘルプメッセージの全文は以下の通りです。

```bash:
usage: rpxy [OPTIONS] --config <FILE>

Options:
  -c, --config <FILE>      Configuration file path like ./config.toml
  -l, --log-dir <LOG_DIR>  Directory for log files. If not specified, logs are printed to stdout.
  -h, --help               Print help
  -V, --version            Print version
```

`--log-dir=<log_dir>`を設定すると、指定したディレクトリにログファイルが作成されます。指定しない場合、ログは標準出力に出力されます。

- `${log_dir}/access.log` - アクセスログ
- `${log_dir}/rpxy.log` - システムおよびエラーログ

以上です!
