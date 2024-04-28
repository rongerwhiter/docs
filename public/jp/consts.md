# 常量

!> ここにはすべての定数が含まれていません。すべての定数をご覧になりたい場合は、[ide-helper](https://github.com/swoole/ide-helper/blob/master/output/swoole/constants.php) を参照するかインストールしてください。

## Swoole

常量 | 役割
---|---
SWOOLE_VERSION | 現在のSwooleバージョン、文字列型、例：1.6.0

## コンストラクタの引数

常量 | 役割
---|---
[SWOOLE_BASE](/learn?id=swoole_base) | Baseモードを使用し、ビジネスコードをReactorプロセスで直接実行します
[SWOOLE_PROCESS](/learn?id=swoole_process) | プロセスモードを使用し、ビジネスコードをWorkerプロセスで実行します

## ソケットタイプ

常量 | 役割
---|---
SWOOLE_SOCK_TCP | TCPソケットを作成します
SWOOLE_SOCK_TCP6 | TCP IPv6ソケットを作成します
SWOOLE_SOCK_UDP | UDPソケットを作成します
SWOOLE_SOCK_UDP6 | UDP IPv6ソケットを作成します
SWOOLE_SOCK_UNIX_DGRAM | UNIX Dgramソケットを作成します
SWOOLE_SOCK_UNIX_STREAM | UNIX Streamソケットを作成します
SWOOLE_SOCK_SYNC | 同期クライアント

## SSL暗号化メソッド

常量 | 役割
---|---
SWOOLE_SSLv3_METHOD | -
SWOOLE_SSLv3_SERVER_METHOD | -
SWOOLE_SSLv3_CLIENT_METHOD | -
SWOOLE_SSLv23_METHOD（デフォルトの暗号化メソッド） | -
SWOOLE_SSLv23_SERVER_METHOD | -
SWOOLE_SSLv23_CLIENT_METHOD | -
SWOOLE_TLSv1_METHOD | -
SWOOLE_TLSv1_SERVER_METHOD | -
SWOOLE_TLSv1_CLIENT_METHOD | -
SWOOLE_TLSv1_1_METHOD | -
SWOOLE_TLSv1_1_SERVER_METHOD | -
SWOOLE_TLSv1_1_CLIENT_METHOD | -
SWOOLE_TLSv1_2_METHOD | -
SWOOLE_TLSv1_2_SERVER_METHOD | -
SWOOLE_TLSv1_2_CLIENT_METHOD | -
SWOOLE_DTLSv1_METHOD | -
SWOOLE_DTLSv1_SERVER_METHOD | -
SWOOLE_DTLSv1_CLIENT_METHOD | -
SWOOLE_DTLS_SERVER_METHOD | -
SWOOLE_DTLS_CLIENT_METHOD | -

!> `SWOOLE_DTLSv1_METHOD`、`SWOOLE_DTLSv1_SERVER_METHOD`、`SWOOLE_DTLSv1_CLIENT_METHOD` は Swoole バージョン >= `v4.5.0` で削除されました。

## SSLプロトコル

常量 | 役割
---|---
SWOOLE_SSL_TLSv1 | -
SWOOLE_SSL_TLSv1_1 | -
SWOOLE_SSL_TLSv1_2 | -
SWOOLE_SSL_TLSv1_3 | -
SWOOLE_SSL_SSLv2 | -
SWOOLE_SSL_SSLv3 | -

!> Swoole バージョン >= `v4.5.4` で使用可能

## ログレベル

常量 | 役割
---|---
SWOOLE_LOG_DEBUG | デバッグログ、カーネル開発用にのみ使用
SWOOLE_LOG_TRACE | トレースログ、システムの問題を追跡するために使用可能。デバッグログは慎重に設定され、重要な情報を伴う
SWOOLE_LOG_INFO | 一般情報、情報の表示のみ
SWOOLE_LOG_NOTICE | 通知情報、システムには再起動やシャットダウンなどの動作がある可能性がある
SWOOLE_LOG_WARNING | 警告情報、システムには問題がある可能性がある
SWOOLE_LOG_ERROR | エラー情報、重要なエラーが発生し、すぐに解決する必要がある
SWOOLE_LOG_NONE | ログ情報を閉じる、ログ情報は表示されない

!> `SWOOLE_LOG_DEBUG`および `SWOOLE_LOG_TRACE` の2種類のログは、Swoole拡張をコンパイルするときに `--enable-debug-log` や `--enable-trace-log` を使用して編集する必要があります。通常のバージョンでは、`log_level = SWOOLE_LOG_TRACE` が設定されていても、この種類のログを印刷することはできません。

## トレースタグ

オンラインで運用されているサービスは常に大量のリクエストを処理しており、下位レベルのログは非常に多く生成されます。 `trace_flags` を使用してトレースログのタグを設定し、一部のトレースログのみを表示できます。 `trace_flags` は `|` または演算子を使用して複数のトレース項目を設定できます。

```php
$serv->set([
	'log_level' => SWOOLE_LOG_TRACE,
	'trace_flags' => SWOOLE_TRACE_SERVER | SWOOLE_TRACE_HTTP2,
]);
```

以下のトレース項目をサポートしており、`SWOOLE_TRACE_ALL` を使用してすべての項目をトレースできます：

* `SWOOLE_TRACE_SERVER`
* `SWOOLE_TRACE_CLIENT`
* `SWOOLE_TRACE_BUFFER`
* `SWOOLE_TRACE_CONN`
* `SWOOLE_TRACE_EVENT`
* `SWOOLE_TRACE_WORKER`
* `SWOOLE_TRACE_REACTOR`
* `SWOOLE_TRACE_PHP`
* `SWOOLE_TRACE_HTTP2`
* `SWOOLE_TRACE_EOF_PROTOCOL`
* `SWOOLE_TRACE_LENGTH_PROTOCOL`
* `SWOOLE_TRACE_CLOSE`
* `SWOOLE_TRACE_HTTP_CLIENT`
* `SWOOLE_TRACE_COROUTINE`
* `SWOOLE_TRACE_REDIS_CLIENT`
* `SWOOLE_TRACE_MYSQL_CLIENT`
* `SWOOLE_TRACE_AIO`
* `SWOOLE_TRACE_ALL`
