# WebSocketサーバー

?> 完全にコルーチン化されたWebSocketサーバーの実装で、[Coroutine\Http\Server](/coroutine/http_server)を継承しています。この下には`WebSocket`プロトコルのサポートがあり、ここではその詳細を述べることはしません。

!> このセクションはv4.4.13以降で利用可能です。
```php
use Swoole\Http\Request;
use Swoole\Http\Response;
use Swoole\WebSocket\CloseFrame;
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/websocket', function (Request $request, Response $ws) {
        $ws->upgrade();
        while (true) {
            $frame = $ws->recv();
            if ($frame === '') {
                $ws->close();
                break;
            } else if ($frame === false) {
                echo 'errorCode: ' . swoole_last_error() . "\n";
                $ws->close();
                break;
            } else {
                if ($frame->data == 'close' || get_class($frame) === CloseFrame::class) {
                    $ws->close();
                    break;
                }
                $ws->push("Hello {$frame->data}!");
                $ws->push("How are you, {$frame->data}?");
            }
        }
    });

    $server->handle('/', function (Request $request, Response $response) {
        $response->end(<<<HTML
    <h1>Swoole WebSocket Server</h1>
    <script>
var wsServer = 'ws://127.0.0.1:9502/websocket';
var websocket = new WebSocket(wsServer);
websocket.onopen = function (evt) {
    console.log("Connected to WebSocket server.");
    websocket.send('hello');
};

websocket.onclose = function (evt) {
    console.log("Disconnected");
};

websocket.onmessage = function (evt) {
    console.log('Retrieved data from server: ' + evt.data);
};

websocket.onerror = function (evt, e) {
    console.log('Error occured: ' + evt.data);
};
</script>
HTML
        );
    });

    $server->start();
});
```
```php
use Swoole\Http\Request;
use Swoole\Http\Response;
use Swoole\WebSocket\CloseFrame;
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/websocket', function (Request $request, Response $ws) {
        $ws->upgrade();
        global $wsObjects;
        $objectId = spl_object_id($ws);
        $wsObjects[$objectId] = $ws;
        while (true) {
            $frame = $ws->recv();
            if ($frame === '') {
                unset($wsObjects[$objectId]);
                $ws->close();
                break;
            } else if ($frame === false) {
                echo 'errorCode: ' . swoole_last_error() . "\n";
                $ws->close();
                break;
            } else {
                if ($frame->data == 'close' || get_class($frame) === CloseFrame::class) {
                    unset($wsObjects[$objectId]);
                    $ws->close();
                    break;
                }
                foreach ($wsObjects as $obj) {
                    $obj->push("Server：{$frame->data}");
                }
            }
        }
    });
    $server->start();
});
```
## 処理フロー

* `$ws->upgrade()`：クライアントに`WebSocket`のハンドシェイクメッセージを送信します
* `while(true)` ループでメッセージの受信と送信を処理する
* `$ws->recv()`：`WebSocket`メッセージフレームを受信する
* `$ws->push()`：対向先にデータフレームを送信する
* `$ws->close()`：接続を閉じる

!> `$ws` は `Swoole\Http\Response` オブジェクトであり、各メソッドの使用方法は以下の参考をご覧ください。
## 方法
### upgrade()

Send a successful `WebSocket` handshake message.

!> Do not use this method in servers with [asynchronous style](/http_server)

```php
Swoole\Http\Response->upgrade(): bool
```
### recv()

`WebSocket`のメッセージを受信します。

!> このメソッドは[非同期スタイル](/http_server)のサーバーでは使用しないでください。`recv`メソッドを呼び出すと、現在のコルーチンが[一時停止](/coroutine?id=协程调度)し、データが到着するのを待ってからコルーチンの実行が再開されます

```php
Swoole\Http\Response->recv(float $timeout = 0): Swoole\WebSocket\Frame | false | string
```

* **返り値**

  * メッセージを正常に受信した場合は`Swoole\WebSocket\Frame`オブジェクトが返されます。詳細は [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) を参照してください
  * 失敗した場合は`false`が返され、エラーコードを取得するには [swoole_last_error()](/functions?id=swoole_last_error) を使用してください
  * 接続が閉じられた場合は空の文字列が返されます
  * 返り値の処理については [ブロードキャストの例](/coroutine/ws_server?id=群发示例) を参照してください
### push()

`WebSocket`データフレームを送信します。

!> このメソッドは、[非同期スタイル](/http_server)のサーバーでは使用しないでください。大きなデータパケットを送信する場合は、書き込み可能な状態をリッスンする必要があり、このため[複数のコルーチン切り替え](/coroutine?id=コルーチンスケジューリング)が発生します。

```php
Swoole\Http\Response->push(string|object $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool
```

* **パラメータ** 

  !> 渡された`$data`が [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) オブジェクトである場合、後続のパラメータは無視され、様々なフレームタイプを送信することがサポートされます

  * **`string|object $data`**

    * **機能**：送信する内容
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $opcode`**

    * **機能**：送信データの形式を指定します 【デフォルトはテキストです。バイナリコンテンツを送信する場合、`$opcode`パラメータを`WEBSOCKET_OPCODE_BINARY`に設定する必要があります】
    * **デフォルト値**：`WEBSOCKET_OPCODE_TEXT`
    * **その他の値**：`WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **機能**：送信が完了したかどうか
    * **デフォルト値**：`true`
    * **その他の値**：`false`
```php
Swoole\Http\Response->close(): bool
```

このメソッドは`TCP`接続を直接切断し、`Close`フレームを送信しないため、`WebSocket\Server::disconnect()`メソッドとは異なります。
接続を切断する前に`push()`メソッドを使用して`Close`フレームを送信し、クライアントにアクティブに通知することができます。

```php
$frame = new Swoole\WebSocket\CloseFrame;
$frame->reason = 'close';
$ws->push($frame);
$ws->close();
```
