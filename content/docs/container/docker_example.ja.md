---
title: Dockerの例
type: docs
prev: /docs/container/docker
weight: 3
sidebar:
  open: true
---

## 基本的な例

以下は、バックエンドアプリケーションとして[`whoami`コンテナ](https://github.com/jwilder/whoami)を提供する`docker-compose`での`rpxy`の基本的な使用例です。

以下のコマンドでコンテナを起動できます。

```bash
% docker-compose up -d -f docker-compose.yml
```

具体的な設定は以下の通りです:

- `rpxy`はポート80と443でリッスン
  - HTTP/1.1、HTTP/2、HTTP/3に対応。
  - HTTPSリダイレクションが有効。
  - TLS証明書はLet's Encryptから自動取得。
- `whoami`はポート8000でリッスン（外部には公開されない）。
- `rpxy`はホスト名とTLS SNIに基づいてリクエストを`whoami`コンテナに転送。
- `rpxy`のログは`./log`ディレクトリに保存。
- `rpxy`のキャッシュは`./cache`ディレクトリに保存。
- `rpxy`のACME証明書と秘密鍵は`./acme_registry`ディレクトリに保存。
- `rpxy`の設定は`./config/rpxy.toml`として保存。

### `docker-compose.yml`

```yaml:docker-compose.yml
services:
  rpxy:
    image: jqtype/rpxy:latest
    container_name: rpxy
    init: true
    ports:
      - 80:80/tcp
      - 443:443/tcp
      - 443:443/udp # http/3
    restart: unless-stopped
    privileged: true
    environment:
      - HOST_USER=ubuntu
      - HOST_UID=1000
      - HOST_GID=1000
      - LOG_LEVEL=debug
      - LOG_TO_FILE=true
    volumes:
      - ./config:/rpxy/config:ro               # for config
      - ./log:/rpxy/log                   # for log
      - ./cache:/tmp/rpxy:rw                   # for cache
      - ./acme_registry:/rpxy/acme_registry:rw # for acme

  whoami:
    image: jwilder/whoami
    container_name: whoami
    restart: always
    logging:
      options:
        max-size: "10m"
        max-file: "3"
    expose: # expose port 8000 for rpxy (not for host and outside)
      - 8000
```

### `./config/rpxy.toml`

```toml:./config/rpxy.toml
########################################
#       rust-rpxy configuration        #
########################################
###################################
#         Global settings         #
###################################
listen_port = 80
listen_port_tls = 443

###################################
#         Backend settings        #
###################################
[application]

[apps."whoami.example.com"]
server_name = 'whoami.example.com'
reverse_proxy = [
  { upstream = [
    { location = 'whoami:8000', tls = false },
  ] }
]
tls = { https_redirection = true, acme = true }


###################################
#      Experimantal settings      #
###################################
[experimental]

[experimental.h3]

[experimental.cache]
cache_dir = '/tmp/rpxy/cache' # default is "./cache" relative to the current working directory

[experimental.acme]
email = "yourname@example.com"
```
