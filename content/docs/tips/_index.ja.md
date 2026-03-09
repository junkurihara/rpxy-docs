---
title: TIPS
type: docs
prev: /docs/
weight: 40
---

## Let's Encryptで発行された秘密鍵の使用

[Let's Encrypt](https://letsencrypt.org/)から証明書と秘密鍵を取得した場合、PKCS1形式の秘密鍵が得られます。そのため、`rpxy`で使用するには取得した秘密鍵をPKCS8形式に変換する必要があります。

最も簡単な方法は`openssl`を使用することです。

```bash
% openssl pkcs8 -topk8 -nocrypt \
    -in your_domain_from_le.key \
    -inform PEM \
    -out your_domain_pkcs8.key.pem \
    -outform PEM
```

## 独自のルートCAで署名されたクライアント証明書を使用したクライアント認証

まず、クライアント証明書を検証するために使用するCA証明書を準備する必要があります。お持ちでない場合は、以下のOpenSSLコマンドでCAキーと証明書を生成できます。*`rustls`はX509v3証明書を受け入れSHA-1を拒否すること、そして`rpxy`は`Subject Key Identifier`と`Authority Key Identifier`のVersion 3拡張フィールドの`KeyID`に依存していることにご注意ください。*

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

## (回避策) `ufw`の背後でDockerを使用したUbuntu 22.04LTSへのデプロイ

基本的に、ポートマッピングオプション（`docker run`の`--publish`や`docker-compose.yml`の`ports`）を使用する場合、Dockerはiptablesを自動管理します。つまり、HTTPS用のポート443 TCP/UDPなどを`ufw`のような管理コマンドで手動で公開する必要はありません。

しかし、`rpxy`で最新のUDPベースプロトコルであるHTTP/3を使用する場合、`ufw`のようなコマンドを使用してHTTPSポートを明示的に公開する必要があることがわかりました。

```bash
% sudo ufw allow 443
% sudo ufw enable
```

ポートを手動管理しない限り、DockerコンテナはTCPベースの接続（HTTP/2以前）のみを受信できます。これは奇妙であり、何らかのバグ（Docker? Ubuntu? その他?）であると考えています。しかし、少なくともUbuntu 22.04LTSでは、上記のように対処する必要があります。

## Webインターフェースによる`rpxy`の管理

Webインターフェースで`rpxy`を管理するには、サードパーティプロジェクト[`Gamerboy59/rpxy-webui`](https://github.com/Gamerboy59/rpxy-webui)をご確認ください。
