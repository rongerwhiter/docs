# ライブラリ

Swooleはv4以降、[Library](https://github.com/swoole/library)モジュールを組み込んでいます。**PHPコードを使用してコア機能を記述**しており、システムの基盤をより安定かつ信頼性の高いものにしています

!> このモジュールはcomposerを使用して単独でインストールすることもできます。単独でインストールする場合は、`php.ini`で`swoole.enable_library=Off`を設定して組み込みライブラリを無効にする必要があります

現在、以下のツールコンポーネントが提供されています：

- [Coroutine\WaitGroup](https://github.com/swoole/library/blob/master/src/core/Coroutine/WaitGroup.php)：並行したコルーチンタスクを待機するためのコンポーネント、[ドキュメント](/coroutine/wait_group)
- [Coroutine\FastCGI](https://github.com/swoole/library/tree/master/src/core/Coroutine/FastCGI)：FastCGIクライアント、[ドキュメント](/coroutine_client/fastcgi)
- [Coroutine\Server](https://github.com/swoole/library/blob/master/src/core/Coroutine/Server.php)：コルーチンサーバ、[ドキュメント](/coroutine/server)
- [Coroutine\Barrier](https://github.com/swoole/library/blob/master/src/core/Coroutine/Barrier.php)：コルーチンバリア、[ドキュメント](/coroutine/barrier)

- [CURL hook](https://github.com/swoole/library/tree/master/src/core/Curl)：CURLのコルーチン化、[ドキュメント](/runtime?id=swoole_hook_curl)
- [Database](https://github.com/swoole/library/tree/master/src/core/Database)：さまざまなデータベース接続プールとオブジェクトプロキシの高度なラッパー、[ドキュメント](/coroutine/conn_pool?id=database)
- [ConnectionPool](https://github.com/swoole/library/blob/master/src/core/ConnectionPool.php)：基本的な接続プール、[ドキュメント](/coroutine/conn_pool?id=connectionpool)
- [Process\Manager](https://github.com/swoole/library/blob/master/src/core/Process/Manager.php)：プロセスマネージャ、[ドキュメント](/process/process_manager)

- [StringObject](https://github.com/swoole/library/blob/master/src/core/StringObject.php)、[ArrayObject](https://github.com/swoole/library/blob/master/src/core/ArrayObject.php)、[MultibyteStringObject](https://github.com/swoole/library/blob/master/src/core/MultibyteStringObject.php)：オブジェクト指向のArrayとStringプログラミング

- [functions](https://github.com/swoole/library/blob/master/src/core/Coroutine/functions.php)：いくつかのコルーチン関数を提供します、[ドキュメント](/coroutine/coroutine?id=関数)
- [Constant](https://github.com/swoole/library/tree/master/src/core/Constant.php)：一般的な設定定数
- [HTTP Status](https://github.com/swoole/library/blob/master/src/core/Http/Status.php)：HTTPステータスコード

## サンプルコード

[Examples](https://github.com/swoole/library/tree/master/examples)
