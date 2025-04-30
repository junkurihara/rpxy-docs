---
title: Docker Examples
type: docs
prev: /docs/container/docker
# next: /docs/container/
weight: 3
sidebar:
  open: true
---

## Basic Example

The following is a basic example of using `rpxy` with `docker-compose`, serving [`whoami` container](https://github.com/jwilder/whoami) as a backend application.

You can run the following commands to start the containers.

```bash
% docker-compose up -d -f docker-compose.yml
```

Specifically, the following is the configuration:

- `rpxy` listens on port 80 and 443
  - HTTP/1.1, HTTP/2, and HTTP/3 are supported.
  - HTTPS redirection is enabled.
  - TLS Certificates are automatically obtained from Let's Encrypt.
- `whoami` listens on port 8000, which is not exposed outside.
- `rpxy` forwards requests to `whoami` container based on the hostname and TLS SNI.
- `rpxy` logs are saved in `./log` directory.
- `rpxy` cache is saved in `./cache` directory.
- `rpxy` ACME certificates and private keys are saved in `./acme_registry` directory.
- `rpxy` configuration is saved as `./config/rpxy.toml`

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
#       rust-rxpy configuration        #
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
