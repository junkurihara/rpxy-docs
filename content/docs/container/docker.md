---
title: Running rpxy in Docker
type: docs
prev: /docs/container/
next: /docs/container/docker_example
weight: 2
sidebar:
  open: true
---

## Environment Variables for Docker

There are several docker-specific environment variables.

| Variable | Description |
| --- | --- |
|`HOST_USER` (default: `user`)| User name executing `rpxy` inside the container.|
|`HOST_UID` (default: `900`)| `UID` of `HOST_USER`.|
|`HOST_GID` (default: `900`)| `GID` of `HOST_USER`|
|`LOG_LEVEL` (default: `info`) | Log level, `debug`, `info`, `warn` or `error` |
|`LOG_TO_FILE` (default: `false`) | Enable logging to the log file `/rpxy/log/rpxy.log` using `logrotate`. You should mount `/rpxy/log` via docker volume option if enabled.|
|`WATCH` (default: `false`) | Activate continuous watching of the config file if true. `true` or `false`|

All you need is to mount your `config.toml` as `/etc/rpxy.toml` and certificates/private keys as you like through the docker volume option. Note that the file path of keys and certificates must be ones in your docker container.

{{< callout type="info" >}}
For `LOG_TO_FILE` option, the log dir and file will be owned by the `HOST_USER` with `HOST_UID:HOST_GID` on the host machine. Hence, `HOST_USER`, `HOST_UID` and `HOST_GID` should be the same as ones of the user who executes the `rpxy` docker container on the host.
{{< /callout >}}

{{< callout type="warning" >}}
 **If `WATCH=true`, You need to mount a directory, e.g., `./rpxy-config/`, including `rpxy.toml` on `/rpxy/config` instead of a file to correctly track file changes**. This is a docker limitation. Even if `WATCH=false`, you can mount the dir onto `/rpxy/config` rather than `/etc/rpxy.toml`. A file mounted on `/etc/rpxy` is prioritized over a dir mounted on `/rpxy/config`.
{{< /callout >}}

See [`docker-compose.yml`](https://github.com/junkurihara/rust-rpxy/blob/develop/docker/docker-compose.yml) or [the following section](/docs/container/docker_example) for examples of docker configuration.

## Set custom port for HTTPS redirection in non-priviliged mode without exposure of well-known ports (80, 443)

For security reasons, you may want to run `rpxy` in a non-privileged mode without exposing well-known ports. In this case, you should specify custom port for HTTPS redirection.

When the container manager maps port A (e.g., 80/443) of the host to port B (e.g., 8080/8443) of the container for http and https, `rpxy` must be configured with port B for `listen_port` and `listen_port_tls`.

However, when you want to set `http_redirection=true` for some backend apps, `rpxy` issues the redirection response 301 with the port B by default, which is not accessible from the outside of the container.

To avoid the above issue, you can set a custom port for the redirection response by specifying `https_redirection_port` in `config.toml`. In this case, port A should be set for `https_redirection_port`, then the redirection response 301 will be issued with the port A.

The following is an example of relevant settings in `config.toml`.

```toml
listen_port = 8080            # `rpxy` in the container listens on port 8080
listen_port_tls = 8443        # `rpxy` in the container listens on port 8443 for TLS
https_redirection_port = 443  # `rpxy` issues the redirection response 301 with port 443
```

## Custom CAs for upstream TLS connections

To add a custom certificate, you must use a non-`webpki` image. Then mount `/usr/local/share/ca-certificates` in the container with your desired CAs each in a file like `myca.crt`. The certificates are accepted in PEM format but file extension must be `crt`.

e.g. `-v rpxy/ca-certificates:/usr/local/share/ca-certificates`
