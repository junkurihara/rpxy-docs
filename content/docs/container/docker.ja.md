---
title: Dockerでのrpxyの実行
type: docs
prev: /docs/container/
next: /docs/container/docker_example
weight: 2
sidebar:
  open: true
---

## Docker用環境変数

Docker固有の環境変数がいくつかあります。

| 変数 | 説明 |
| --- | --- |
|`HOST_USER` (デフォルト: `user`)| コンテナ内で`rpxy`を実行するユーザー名。|
|`HOST_UID` (デフォルト: `900`)| `HOST_USER`の`UID`。|
|`HOST_GID` (デフォルト: `900`)| `HOST_USER`の`GID`。|
|`LOG_LEVEL` (デフォルト: `info`) | ログレベル。`debug`、`info`、`warn`または`error`。 |
|`LOG_TO_FILE` (デフォルト: `false`) | `/rpxy/log/rpxy.log`と`/rpxy/log/access.log`へのファイルログを`logrotate`を使用して有効にします。有効にする場合はDockerボリュームオプションで`/rpxy/log`をマウントしてください。|

必要なのは、`config.toml`を`/etc/rpxy.toml`としてマウントし、証明書/秘密鍵をDockerボリュームオプションで任意の場所にマウントするだけです。鍵と証明書のファイルパスはDockerコンテナ内のパスである必要があります。

{{< callout type="info" >}}
`LOG_TO_FILE`オプションについて、ログディレクトリとファイルはホストマシン上で`HOST_UID:HOST_GID`の`HOST_USER`が所有します。そのため、`HOST_USER`、`HOST_UID`、`HOST_GID`はホストで`rpxy`のDockerコンテナを実行するユーザーと同じにする必要があります。
{{< /callout >}}

{{< callout type="warning" >}}
**ファイルの変更を動的に追跡するには、ファイルではなく`./rpxy-config/`などのディレクトリを`/rpxy/config`にマウントする必要があります**。これはDockerの制限です。`/etc/rpxy.toml`ではなく`/rpxy/config`にディレクトリをマウントできます。`/etc/rpxy`にマウントされたファイルは`/rpxy/config`にマウントされたディレクトリよりも優先されます。
{{< /callout >}}

例については[`docker-compose.yml`](https://github.com/junkurihara/rust-rpxy/blob/develop/docker/docker-compose.yml)または[次のセクション](/docs/container/docker_example)を参照してください。

## 非特権モードでwell-knownポート(80, 443)を公開せずにHTTPSリダイレクション用カスタムポートを設定する

セキュリティ上の理由から、well-knownポートを公開せずに非特権モードで`rpxy`を実行したい場合があります。その場合、HTTPSリダイレクション用のカスタムポートを指定する必要があります。

コンテナマネージャーがホストのポートA（例: 80/443）をコンテナのポートB（例: 8080/8443）にマッピングする場合、`rpxy`は`listen_port`と`listen_port_tls`にポートBを設定する必要があります。

しかし、一部のバックエンドアプリに`http_redirection=true`を設定したい場合、`rpxy`はデフォルトでポートBでリダイレクションレスポンス301を発行しますが、これはコンテナ外からアクセスできません。

この問題を回避するため、`config.toml`で`https_redirection_port`を指定してリダイレクションレスポンス用のカスタムポートを設定できます。この場合、`https_redirection_port`にポートAを設定すると、リダイレクションレスポンス301がポートAで発行されます。

関連する設定の例は以下の通りです。

```toml
listen_port = 8080            # コンテナ内の`rpxy`はポート8080でリッスン
listen_port_tls = 8443        # コンテナ内の`rpxy`はポート8443でTLSをリッスン
https_redirection_port = 443  # `rpxy`はポート443でリダイレクションレスポンス301を発行
```

## アップストリームTLS接続用のカスタムCA

カスタム証明書を追加するには、非`webpki`イメージを使用する必要があります。コンテナ内の`/usr/local/share/ca-certificates`に、`myca.crt`のようなファイルで希望するCAをマウントしてください。証明書はPEM形式で受け入れられますが、ファイル拡張子は`crt`である必要があります。

例: `-v rpxy/ca-certificates:/usr/local/share/ca-certificates`
