---
title: Command Line Options
type: docs
prev: /docs/guide/installation
next: /docs/guide/basics
weight: 2
sidebar:
  open: true
---

`rpxy` always requires a configuration file in TOML format, e.g., [`config.toml` on GitHub repo](https://github.com/junkurihara/rust-rpxy/blob/develop/config-example.toml).

{{< callout type="info" >}}
We will introduce some typical configurations in the [Basic Setups](/docs/guide/basics) and [Advanced Usage](/docs/guide/advanced) sections.
The detailed configuration options are described in the [Configuration Options](/docs/configuration) section.
{{< /callout >}}

You can run `rpxy` with a configuration file like

```bash
% ./path/to/rpxy --config config.toml
```

If you specify `-w` option along with the config file path, `rpxy` tracks the change of `config.toml` in the real-time manner and apply the change immediately without restarting the process.

The full help messages are given follows.

```bash:
usage: rpxy [OPTIONS] --config <FILE>

Options:
  -c, --config <FILE>      Configuration file path like ./config.toml
  -l, --log-dir <LOG_DIR>  Directory for log files. If not specified, logs are printed to stdout.
  -h, --help               Print help
  -V, --version            Print version
```

If you set `--log-dir=<log_dir>`, the log files are created in the specified directory. Otherwise, the log is printed to stdout.

- `${log_dir}/access.log` for access log
- `${log_dir}/rpxy.log` for system and error log

That's all!
