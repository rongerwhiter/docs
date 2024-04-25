# 関数のエイリアスのまとめ

## コルーチンの短縮名

コルーチン関連の`API`の名前を簡略化します。`php.ini`で`swoole.use_shortname=On/Off`を設定して、短縮名の有効化/無効化を行うことができます。デフォルトは有効です。

すべての`Swoole\Coroutine`接頭辞のクラス名が`Co`にマッピングされます。以下にいくつかの追加のマッピングがあります：

### コルーチンの作成

```php
//Swoole\Coroutine::createはgo関数と同等です
go(function () {
	Co::sleep(0.5);
	echo 'hello';
});
go('test');
go([$object, 'method']);
```

### チャネル操作

```php
//Coroutine\Channelはchanに簡略化できます
$c = new chan(1);
$c->push($data);
$c->pop();
```

### 遅延実行

```php
//Swoole\Coroutine::deferはdeferをそのまま使用できます
defer(function () use ($db) {
    $db->close();
});
```

## 短縮名のメソッド

!> 以下の方法は`go`と`defer`。Swoole バージョン >= `v4.6.3` で使用可能です。

```php
use function Swoole\Coroutine\go;
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\defer;

run(function () {
    defer(function () {
        echo "co1 end\n";
    });
    sleep(1);
    go(function () {
        usleep(100000);
        defer(function () {
            echo "co2 end\n";
        });
        echo "co2\n";
    });
    echo "co1\n";
});
```

## コルーチンシステムAPI

`4.4.4`バージョンでは、システム操作関連のコルーチン`API`が`Swoole\Coroutine`クラスから`Swoole\Coroutine\System`クラスに移動されました。新しいモジュールとして独立しています。下位互換性のため、低レベルでは、引き続き`Coroutine`クラス上のエイリアスメソッドが維持されています。

* 例えば `Swoole\Coroutine::sleep`は`Swoole\Coroutine\System::sleep`に対応しています
* 例えば `Swoole\Coroutine::fgets`は`Swoole\Coroutine\System::fgets`に対応しています

## クラスの短いエイリアスマッピング

!> 名前空間スタイルを使用することが推奨されています。

| アンダースコア付きクラス名スタイル | 名前空間スタイル               |
| --------------------------- | --------------------------- |
| swoole_server               | Swoole\Server               |
| swoole_client               | Swoole\Client               |
| swoole_process              | Swoole\Process              |
| swoole_timer                | Swoole\Timer                |
| swoole_table                | Swoole\Table                |
| swoole_lock                 | Swoole\Lock                 |
| swoole_atomic               | Swoole\Atomic               |
| swoole_atomic_long          | Swoole\Atomic\Long          |
| swoole_buffer               | Swoole\Buffer               |
| swoole_redis                | Swoole\Redis                |
| swoole_error                | Swoole\Error                |
| swoole_event                | Swoole\Event                |
| swoole_http_server          | Swoole\Http\Server          |
| swoole_http_client          | Swoole\Http\Client          |
| swoole_http_request         | Swoole\Http\Request         |
| swoole_http_response        | Swoole\Http\Response        |
| swoole_websocket_server     | Swoole\WebSocket\Server     |
| swoole_connection_iterator  | Swoole\Connection\Iterator  |
| swoole_exception            | Swoole\Exception            |
| swoole_http2_request        | Swoole\Http2\Request        |
| swoole_http2_response       | Swoole\Http2\Response       |
| swoole_process_pool         | Swoole\Process\Pool         |
| swoole_redis_server         | Swoole\Redis\Server         |
| swoole_runtime              | Swoole\Runtime              |
| swoole_server_port          | Swoole\Server\Port          |
| swoole_server_task          | Swoole\Server\Task          |
| swoole_table_row            | Swoole\Table\Row            |
| swoole_timer_iterator       | Swoole\Timer\Iterator       |
| swoole_websocket_closeframe | Swoole\Websocket\Closeframe |
| swoole_websocket_frame      | Swoole\Websocket\Frame      |
