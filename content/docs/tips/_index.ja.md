---
title: TIPS
type: docs
prev: /docs/
weight: 40
---

## Let's Encrypt の秘密鍵を使う

[Let's Encrypt](https://letsencrypt.org/) で取得した秘密鍵は PKCS1 形式です。`rpxy` で使うには、PKCS8 形式へ変換する必要があります。

もっとも簡単なのは `openssl` を使う方法です。

```bash
% openssl pkcs8 -topk8 -nocrypt \
    -in your_domain_from_le.key \
    -inform PEM \
    -out your_domain_pkcs8.key.pem \
    -outform PEM
```

## 独自ルート CA で署名したクライアント証明書を使う

まず、クライアント証明書の検証に使う CA 証明書を用意します。まだない場合は、以下の OpenSSL コマンドで CA キーと証明書を生成できます。*`rustls` は X509v3 証明書を受け入れ、SHA-1 は拒否します。また `rpxy` は、Version 3 拡張フィールド内の `Subject Key Identifier` と `Authority Key Identifier` の `KeyID` を参照します。*

1. `secp256v1`のCAキー、CSRを生成し、`config.toml`の各サーバーアプリの`tls.client_ca_cert_path`に設定するCA証明書を生成します。

  ```bash
  % openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:prime256v1 -out client.ca.key

  % openssl req -new -key client.ca.key -out client.ca.csr
  ...
  -----
  Country Name (2 letter code) []: ...
  State or Province Name (full name) []: ...
  Locality Name (eg, city) []: ...
  Organization Name (eg, company) []: ...
  Organizational Unit Name (eg, section) []: ...
  Common Name (eg, fully qualified host name) []: <CNは入力しないでください>
  Email Address []: ...

  % openssl x509 -req -days 3650 -sha256 -in client.ca.csr -signkey client.ca.key -out client.ca.crt -extfile client.ca.ext
  ```

2. `secp256v1`のクライアントキーとCAキーで署名された証明書を生成します。

  ```bash
  % openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:prime256v1 -out client.key

  % openssl req -new -key client.key -out client.csr
  ...
  -----
  Country Name (2 letter code) []:
  State or Province Name (full name) []:
  Locality Name (eg, city) []:
  Organization Name (eg, company) []:
  Organizational Unit Name (eg, section) []:
  Common Name (eg, fully qualified host name) []: <CNは入力しないでください>
  Email Address []:

  % openssl x509 -req -days 365 -sha256 -in client.csr -CA client.ca.crt -CAkey client.ca.key -CAcreateserial -out client.crt -extfile client.ext
  ```

  これでクライアントキー`client.key`と証明書`client.crt`（バージョン3）が得られました。`pfx` (`p12`)ファイルは以下のように取得できます。

  ```bash
  % openssl pkcs12 -export -inkey client.key -in client.crt -certfile client.ca.crt -out client.pfx
  ```

  macOSでは、`OpenSSL 3.0.6`で生成された`pfx`はmacOSのキーチェーンアクセスにインポートできないことにご注意ください。サンプルの`pfx`は`OpenSSL`の代わりに`LibreSSL 2.8.3`を使用して生成しました。

  すべてのサンプル証明書ファイルは`./example-certs/`ディレクトリにあります。

## 回避策: `ufw` 配下で Docker を使う Ubuntu 22.04 LTS へのデプロイ

通常、ポートマッピング（`docker run` の `--publish` や `docker-compose.yml` の `ports`）を使えば、Docker が iptables を自動で管理します。そのため、HTTPS 用の 443/TCP や 443/UDP を `ufw` のようなツールで手動開放する必要はありません。

ただし、`rpxy` で UDP ベースの HTTP/3 を使う場合は、`ufw` などで HTTPS ポートを明示的に開ける必要がありました。

```bash
% sudo ufw allow 443
% sudo ufw enable
```

手動でポートを管理しない限り、Docker コンテナは TCP ベースの接続、つまり HTTP/2 以前しか受け付けられませんでした。挙動としては不自然で、Docker か Ubuntu 側の問題の可能性がありますが、少なくとも Ubuntu 22.04 LTS では上記の対処が必要です。

## Web インターフェースで `rpxy` を管理する

Web インターフェースで `rpxy` を管理したい場合は、サードパーティ製の [`Gamerboy59/rpxy-webui`](https://github.com/Gamerboy59/rpxy-webui) を参照してください。
