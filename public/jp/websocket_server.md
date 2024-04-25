# Swoole\WebSocket\Server

内蔵の`WebSocket`サポートを使用して、わずか数行の`PHP`コードで非同期IOのマルチプロセス`WebSocket`サーバーを作成できます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});

$server->on('close', function ($server, $fd) {
    echo "client {$fd} closed\n";
});

$server->start();
```

* **クライアント**

  * `Chrome/Firefox/`などの高バージョンの`IE/Safari`などのブラウザには、`JS`言語の`WebSocket`クライアントが組み込まれています
  * 微信小プログラム開発フレームワークには、`WebSocket`クライアントが組み込まれています
  * [非同期IO](/learn?id=同步io异步io) の`PHP`プログラムでは [Swoole\Coroutine\Http](/coroutine_client/http_client) を`WebSocket`クライアントとして使用できます
  * `Apache/PHP-FPM`またはその他の同期ブロッキングな`PHP`プログラムでは、[同期WebSocketクライアント](https://github.com/matyhtf/framework/blob/master/libs/Swoole/Client/WebSocket.php) を提供する`swoole/framework`を使用できます
  * `WebSocket`クライアントではない場合、`WebSocket`サーバーと通信できません

* **WebSocketクライアントを判断する方法**

使用している接続情報を取得する [以下の例](/server/methods?id=getclientinfo) を使用して、返される配列に [websocket_status](/websocket_server?id=连接状态) が含まれているかどうかを確認して、クライアントが`WebSocket`クライアントかどうかを判断できます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    $client = $server->getClientInfo($frame->fd);
    // または $client = $server->connection_info($frame->fd);
    if (isset($client['websocket_status'])) {
        echo "WebSocket接続です";
    } else {
        echo "WebSocket接続ではありません";
    }
});
```
## イベント

?> `WebSocket`サーバーは、[Swoole\Server](/server/methods)と[Swoole\Http\Server](/http_server)の基本クラスのコールバック関数を受け入れる以外に、追加で`4`つのコールバック関数を設定することができます。そのうち：

* `onMessage`コールバック関数は必須です
* `onOpen`、`onHandShake`、`onBeforeHandShakeResponse`（Swoole5で提供されるイベント）コールバック関数はオプションです
### onBeforeHandshakeResponse

!> Swoole のバージョン >= `v5.0.0` で使用可能

?> **`WebSocket` 接続確立前に発生します。カスタムハンドシェイク処理が不要であるが、応答ヘッダーに一部の`http header`情報を設定したい場合は、このイベントを呼び出すことができます。**

```php
onBeforeHandshakeResponse(Swoole\Http\Request $request, Swoole\Http\Response $response);
```
### onHandShake

?> **`WebSocket`接続の確立後にハンドシェイクが行われます。`WebSocket`サーバーは自動的にハンドシェイクプロセスを行いますが、ユーザーが独自のハンドシェイク処理を行いたい場合は、`onHandShake`イベントコールバック関数を設定できます。**

```php
onHandShake(Swoole\Http\Request $request, Swoole\Http\Response $response);
```

* **ヒント**

  * `onHandShake`イベントコールバックはオプションです
  * `onHandShake`コールバック関数を設定すると`onOpen`イベントは発生しません。アプリケーションコードで`onOpen`ロジックを自分で処理する必要があり、`$server->defer`を使用して`onOpen`ロジックを呼び出すことができます
  * `onHandShake`内で [response->status()](/http_server?id=status) を呼び出してステータスコードを`101`に設定し、[response->end()](/http_server?id=end) を呼び出してレスポンスする必要があります。そうしないとハンドシェイクが失敗します。
  * 組み込みのハンドシェイクプロトコルは`Sec-WebSocket-Version: 13`ですが、古いバージョンのブラウザではハンドシェイクを自分で実装する必要があります

* **注意**

!> カスタムハンドシェイク処理が必要な場合にのみ、このコールバック関数を設定してください。ハンドシェイクプロセスを「カスタマイズ」する必要がない場合は、このコールバックを設定しないで、`Swoole`のデフォルトハンドシェイクを使用してください。以下は、カスタム`handshake`イベントコールバック関数に必要な内容です：

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // print_r( $request->header );
    // if (カスタム条件を満たさない場合、endを返し、falseを返してハンドシェイクを失敗させる) {
    //    $response->end();
    //     return false;
    // }

    // WebSocketのハンドシェイク接続アルゴリズムの検証
    $secWebSocketKey = $request->header['sec-websocket-key'];
    $patten = '#^[+/0-9A-Za-z]{21}[AQgw]==$#';
    if (0 === preg_match($patten, $secWebSocketKey) || 16 !== strlen(base64_decode($secWebSocketKey))) {
        $response->end();
        return false;
    }
    echo $request->header['sec-websocket-key'];
    $key = base64_encode(
        sha1(
            $request->header['sec-websocket-key'] . '258EAFA5-E914-47DA-95CA-C5AB0DC85B11',
            true
        )
    );

    $headers = [
        'Upgrade' => 'websocket',
        'Connection' => 'Upgrade',
        'Sec-WebSocket-Accept' => $key,
        'Sec-WebSocket-Version' => '13',
    ];

    // WebSocket connection to 'ws://127.0.0.1:9502/'
    // failed: Error during WebSocket handshake:
    // Response must not include 'Sec-WebSocket-Protocol' header if not present in request: websocket
    if (isset($request->header['sec-websocket-protocol'])) {
        $headers['Sec-WebSocket-Protocol'] = $request->header['sec-websocket-protocol'];
    }

    foreach ($headers as $key => $val) {
        $response->header($key, $val);
    }

    $response->status(101);
    $response->end();
});
```

!> `onHandShake`コールバック関数を設定すると`onOpen`イベントは発生しません。アプリケーションコードで`onOpen`ロジックを自分で処理する必要があり、`$server->defer`を使用して`onOpen`ロジックを呼び出すことができます

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // ハンドシェイク内容の省略
    $response->status(101);
    $response->end();

    global $server;
    $fd = $request->fd;
    $server->defer(function () use ($fd, $server)
    {
      echo "Client connected\n";
      $server->push($fd, "hello, welcome\n");
    });
});
```
### onOpen

?> **This function is called when the WebSocket client establishes a connection with the server and completes the handshake.**

```php
onOpen(Swoole\WebSocket\Server $server, Swoole\Http\Request $request);
```

* **Tips**

    * `$request` is an [HTTP](/http_server?id=httprequest) request object that contains the handshake request information sent by the client.
    * In the `onOpen` event function, you can use [push](/websocket_server?id=push) to send data to the client or [close](/server/methods?id=close) to close the connection.
    * The `onOpen` event callback is optional.
### onMessage

?> **この関数は、サーバーがクライアントからのデータフレームを受信したときにコールバックされます。**

```php
onMessage(Swoole\WebSocket\Server $server, Swoole\WebSocket\Frame $frame)
```

* **ヒント**

  * `$frame` は[Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)オブジェクトであり、クライアントからのデータフレーム情報が含まれています。
  * `onMessage`コールバックは設定する必要があります。設定しないとサーバーは起動できません。
  * クライアントが送信した`ping`フレームは`onMessage`をトリガーしません。下層で自動的に`pong`パケットが返信され、[open_websocket_ping_frame](/websocket_server?id=open_websocket_ping_frame)パラメータを設定して手動で処理することもできます。

!> `$frame->data` がテキスト形式の場合、エンコーディングは必ず`UTF-8`である必要があります。これは`WebSocket`プロトコルで規定されています。
### onRequest

?> `Swoole\WebSocket\Server`は[Swoole\Http\Server](/http_server)を継承しているため、`Http\Server`が提供するすべての`API`や設定を使用できます。[Swoole\Http\Server](/http_server)セクションを参照してください。

* [onRequest](/http_server?id=on)コールバックが設定されていると、`WebSocket\Server`は`HTTP`サーバーとしても機能します
* [onRequest](/http_server?id=on)コールバックが設定されていない場合、`WebSocket\Server`は`HTTP`リクエストを受信すると`HTTP 400`のエラーページを返します
* `HTTP`を受信してすべての`WebSocket`をトリガーしたい場合は、スコープの問題に注意する必要があります。手続き型では`global`を使用して`Swoole\WebSocket\Server`を参照し、オブジェクト指向では`Swoole\WebSocket\Server`をメンバープロパティに設定できます
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});
$server->on('close', function ($server, $fd) {
    echo "client {$fd} closed\n";
});
$server->on('request', function (Swoole\Http\Request $request, Swoole\Http\Response $response) {
    global $server; // 调用外部的server
    // $server->connections 遍历所有websocket连接用户的fd，给所有用户推送
    foreach ($server->connections as $fd) {
        // 需要先判断是否是正确的websocket连接，否则有可能会push失败
        if ($server->isEstablished($fd)) {
            $server->push($fd, $request->get['message']);
        }
    }
});
$server->start();
```
```php
class WebSocketServer
{
    public $server;

    public function __construct()
    {
        $this->server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
        $this->server->on('open', function (Swoole\WebSocket\Server $server, $request) {
            echo "server: handshake success with fd{$request->fd}\n";
        });
        $this->server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
            echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
            $server->push($frame->fd, "this is server");
        });
        $this->server->on('close', function ($ser, $fd) {
            echo "client {$fd} closed\n";
        });
        $this->server->on('request', function ($request, $response) {
            // Receive the http request and get the value of the 'message' parameter from get, then push it to the users
            // $this->server->connections traverse all websocket connection users' fds, push to all users
            foreach ($this->server->connections as $fd) {
                // Need to check if it is a correct websocket connection first, otherwise push may fail
                if ($this->server->isEstablished($fd)) {
                    $this->server->push($fd, $request->get['message']);
                }
            }
        });
        $this->server->start();
    }
}

new WebSocketServer();
```
### onDisconnect

?> **WebSocketコネクションではない場合にのみこのイベントがトリガーされます。**

!> Swooleのバージョン >= `v4.7.0` で利用可能

```php
onDisconnect(Swoole\WebSocket\Server $server, int $fd)
```

!> `onDisconnect` イベントコールバックが設定されている場合、非WebSocketリクエストまたは [onRequest](/websocket_server?id=onrequest) で `$response->close()` メソッドが呼び出された場合に、`onDisconnect` コールバックがトリガーされます。一方、[onRequest](/websocket_server?id=onrequest) イベント内で正常に終了された場合、`onClose` または `onDisconnect` イベントは呼び出されません。
## 方法

`Swoole\WebSocket\Server`は [Swoole\Server](/server/methods) のサブクラスであり、したがって`Server`のすべてのメソッドを呼び出すことができます。

`WebSocket`サーバーからクライアントにデータを送信する場合は、`Swoole\WebSocket\Server::push`メソッドを使用する必要があります。このメソッドは`WebSocket`プロトコルでデータをパッケージ化します。一方、 [Swoole\Server->send()](/server/methods?id=send) メソッドは元の`TCP`送信インターフェースです。

[Swoole\WebSocket\Server->disconnect()](/websocket_server?id=disconnect)メソッドを使用すると、サーバーからアクティブな`WebSocket`接続を切断できます。[閉じる状態コード](/websocket_server?id=websocket关闭帧状态码)(`WebSocket`プロトコルによると、利用可能な状態コードは10進整数で、`1000`または`4000-4999`の値を取ることができます)および閉じる理由（`utf-8`エンコーディングで、バイト長が`125`を超えない文字列を使用）を指定できます。状態コードが指定されていない場合は`1000`、閉じる理由は空です。
### push

?> **`WebSocket`クライアントへデータをプッシュし、サイズは最大`2M`にする必要があります。**

```php
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool

// v4.4.12のバージョンでは、flagsパラメータに変更されました
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): bool
```

* **パラメータ**

  * **`int $fd`**

    * **機能**: クライアント接続の`ID`【指定された`$fd`が`WebSocket`クライアントに対応していないTCP接続の場合、送信に失敗します】
    * **デフォルト値**: なし
    * **その他の値**: なし

  * **`Swoole\WebSocket\Frame|string $data`**

    * **機能**: 送信するデータの内容
    * **デフォルト値**: なし
    * **その他の値**: なし

  !> Swooleバージョン >= v4.2.0 の場合、渡された`$data`が [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) オブジェクトの場合、その後続するパラメータは無視されます

  * **`int $opcode`**

    * **機能**: 送信するデータの形式を指定【デフォルトはテキスト。バイナリコンテンツを送信する場合は`WEBSOCKET_OPCODE_BINARY`に設定する必要があります】
    * **デフォルト値**: `WEBSOCKET_OPCODE_TEXT`
    * **その他の値**: `WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **機能**: 送信が完了したかどうか
    * **デフォルト値**: `true`
    * **その他の値**: `false`

* **戻り値**

  * 成功した場合は`true`、失敗した場合は`false`

!> `v4.4.12`以降、`finish`パラメータ（`bool`型）は`flags`パラメータ（`int`型）に変更され、`WebSocket`の圧縮をサポートするために`finish`は`SWOOLE_WEBSOCKET_FLAG_FIN`値が`1`に対応します。元の`bool`型の値は`int`型に暗黙的に変換されます。 また、圧縮`flag`は`SWOOLE_WEBSOCKET_FLAG_COMPRESS`です。

!> [BASE モード](/learn?id=baseモードの限制：) では、プロセス間での `push` データ送信はサポートされていません。
### exist

?> **判断`WebSocket`クライアントが存在し、`Active`状態であるかどうかを確認します。**

!> `v4.3.0`以降、この`API`は接続が存在するかどうかを判断するためにのみ使用され、`isEstablished`を使用して`WebSocket`接続であるかどうかを判断してください。

```php
Swoole\WebSocket\Server->exist(int $fd): bool
```

* **戻り値**

  * 接続が存在し、`WebSocket`ハンドシェイクが完了している場合は`true`が返されます。
  * 接続が存在しないか、ハンドシェイクが完了していない場合は`false`が返されます。
### パック

?> **WebSocketメッセージをパックします。**

```php
Swoole\WebSocket\Server::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true, bool $mask = false): string

// v4.4.12バージョンでは、flagsパラメータに変更されました
Swoole\WebSocket\Server::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): string

Swoole\WebSocket\Frame::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): string
```

* **パラメータ** 

  * **`Swoole\WebSocket\Frame|string $data $data`**

    * **機能**：メッセージの内容
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $opcode`**

    * **機能**：送信するデータの形式を指定します【デフォルトはテキストです。バイナリコンテンツを送信する場合，`$opcode`パラメータを`WEBSOCKET_OPCODE_BINARY`に設定する必要があります】
    * **デフォルト値**：`WEBSOCKET_OPCODE_TEXT`
    * **その他の値**：`WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **機能**：フレームが完了したかどうか
    * **デフォルト値**：なし
    * **その他の値**：なし

    !> `v4.4.12`以降、`finish`パラメータ（`bool`型）が`flags`パラメータ（`int`型）に変更されました。これにより`WebSocket`圧縮をサポートし、`finish`は`SWOOLE_WEBSOCKET_FLAG_FIN`の値`1`に対応し、元の`bool`型の値は暗黙的に`int`型に変換されます。この変更は下位互換性に影響しません。

  * **`bool $mask`**

    * **機能**：マスクを設定するかどうか【`v4.4.12`でこのパラメータは削除されました】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * パックされた`WebSocket`データパケットが返され、[send()](/server/methods?id=send) メソッドを使用して対向に送信できます。

* **例**

```php
$ws = new Swoole\Server('127.0.0.1', 9501, SWOOLE_BASE);

$ws->set(array(
    'log_file' => '/dev/null'
));

$ws->on('WorkerStart', function (\Swoole\Server $serv) {
});

$ws->on('receive', function ($serv, $fd, $threadId, $data) {
    $sendData = "HTTP/1.1 101 Switching Protocols\r\n";
    $sendData .= "Upgrade: websocket\r\nConnection: Upgrade\r\nSec-WebSocket-Accept: IFpdKwYy9wdo4gTldFLHFh3xQE0=\r\n";
    $sendData .= "Sec-WebSocket-Version: 13\r\nServer: swoole-http-server\r\n\r\n";
    $sendData .= Swoole\WebSocket\Server::pack("hello world\n");
    $serv->send($fd, $sendData);
});

$ws->start();
```
### unpack

?> **WebSocketデータフレームを解析します。**

```php
Swoole\WebSocket\Server::unpack(string $data): Swoole\WebSocket\Frame|false;
```

* **パラメータ**

  * **`string $data`**

    * **機能**：メッセージの内容
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * 解析に失敗した場合は`false`が返され、解析に成功した場合は[Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)オブジェクトが返されます
### 切断

?> **WebSocket クライアントにアクティブにクローズフレームを送信し、接続を切断します。**

!> Swooleバージョン >= `v4.0.3` で利用可能

```php
Swoole\WebSocket\Server->disconnect(int $fd, int $code = SWOOLE_WEBSOCKET_CLOSE_NORMAL, string $reason = ''): bool
```

* **パラメータ** 

  * **`int $fd`**

    * **機能**：クライアント接続の`ID`【指定された`$fd`がWebSocketクライアントに対応していないTCP接続の場合、送信に失敗する可能性があります】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $code`**

    * **機能**：接続を閉じる状態コード【`RFC6455` に基づくアプリケーション終了の接続状態コードは、`1000` または `4000` から `4999` の間の値となります】
    * **デフォルト値**：`SWOOLE_WEBSOCKET_CLOSE_NORMAL`
    * **その他の値**：なし

  * **`string $reason`**

    * **機能**：接続を閉じる理由【`UTF-8` 形式の文字列で、バイト長は `125` を超えない】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * 送信が成功した場合は`true`が返され、失敗したか、状態コードが不正な場合は`false`が返されます。
### isEstablished

?> **Check if the connection is a valid `WebSocket` client connection.**

?> This function is different from the `exist` method, which only determines if it is a `TCP` connection and cannot determine if it is a completed handshake of a `WebSocket` client.

```php
Swoole\WebSocket\Server->isEstablished(int $fd): bool
```

* **Parameters** 

  * **`int $fd`**

    * **Description**：The client connection's `ID`【If the specified `$fd` does not correspond to a `WebSocket` client, the check will fail】
    * **Default**：None
    * **Other values**：None

* **Return Value**

  * Returns `true` if it is a valid connection, otherwise returns `false`
## Websocketデータフレームクラス
### Swoole\WebSocket\Frame

`v4.2.0`バージョンでは、サーバーとクライアントが[Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)オブジェクトを送信できるようになりました。  
`v4.4.12`バージョンでは、`flags`プロパティが追加され、`WebSocket`圧縮フレームをサポートし、新しいサブクラス[Swoole\WebSocket\CloseFrame](/websocket_server?id=swoolewebsocketcloseframe)が追加されました。

通常の`frame`オブジェクトには以下の属性があります。

定数 | 説明 
---|--- 
fd |  クライアントの`socket id`。`$server->push`を使用してデータをプッシュする際に必要です。    
data | データ内容。テキストコンテンツまたはバイナリデータのいずれかで、`opcode`の値に基づいて判断できます。   
opcode | `WebSocket`の[データフレームのタイプ](/websocket_server?id=データフレームタイプ)。`WebSocket`プロトコルの仕様書を参照してください。    
finish | データフレームが完全かどうかを示します。1つの`WebSocket`リクエストは複数のデータフレームに分割されて送信される場合があります（データフレームの自動統合が既に実装されているため、受信したデータフレームが不完全である心配はいりません）  

このクラスには[Swoole\WebSocket\Frame::pack()](/websocket_server?id=pack)および[Swoole\WebSocket\Frame::unpack()](/websocket_server?id=unpack)が付属しており、`websocket`メッセージのパッキングおよびアンパッキングに使用されます。引数の説明は`Swoole\WebSocket\Server::pack()`および`Swoole\WebSocket\Server::unpack()`と同じです。
### Swoole\WebSocket\CloseFrame

A regular `close frame` object has the following attributes

Constant | Description
---|---
opcode | [Data frame type](/websocket_server?id=data-frame-type) for `WebSocket`, refer to the `WebSocket` protocol standard document    
code | [Close frame status code](/websocket_server?id=websocket-disconnection-status-codes) for `WebSocket`, refer to the [error codes defined in the websocket protocol](https://developer.mozilla.org/zh-CN/docs/Web/API/CloseEvent)    
reason | Closing reason, if not explicitly provided, it is empty

If the server needs to receive a `close frame`, it must enable the [open_websocket_close_frame](/websocket_server?id=open_websocket_close_frame) parameter with `$server->set`. 

Translated text:
### Swoole\WebSocket\CloseFrame

通常の`close frame`オブジェクトには以下の属性があります

定数 | 説明
---|---
opcode | `WebSocket`の[データフレームタイプ](/websocket_server?id=data-frame-type)、`WebSocket`プロトコルの標準ドキュメントを参照してください    
code | `WebSocket`の[クローズフレームステータスコード](/websocket_server?id=websocket-disconnection-status-codes)、[websocketプロトコルで定義されたエラーコード](https://developer.mozilla.org/zh-CN/docs/Web/API/CloseEvent)を参照してください    
reason | クローズ理由、明示的に指定されていない場合は空です

サーバーが`close frame`を受信する必要がある場合、`$server->set`で[open_websocket_close_frame](/websocket_server?id=open_websocket_close_frame)パラメータを有効にする必要があります。
```python
PI = 3.14159
SPEED_OF_LIGHT = 299792458
GRAVITATIONAL_CONSTANT = 6.67430e-11
```

これらの定数は数学や物理学で一般的に使用される定数です。
### データフレームのタイプ

定数 | 値 | 説明
---|---|---
WEBSOCKET_OPCODE_TEXT | 0x1 | UTF-8テキストデータ
WEBSOCKET_OPCODE_BINARY | 0x2 | バイナリデータ
WEBSOCKET_OPCODE_CLOSE | 0x8 | クローズフレームのタイプデータ
WEBSOCKET_OPCODE_PING | 0x9 | pingタイプのデータ
WEBSOCKET_OPCODE_PONG | 0xa | pongタイプのデータ
### 接続状態

定数 | 値 | 説明
---|---|---
WEBSOCKET_STATUS_CONNECTION | 1 | ハンドシェイク待機中の接続
WEBSOCKET_STATUS_HANDSHAKE | 2 | ハンドシェイク中
WEBSOCKET_STATUS_ACTIVE | 3 | ハンドシェイク成功、ブラウザからのデータフレームを待機中
WEBSOCKET_STATUS_CLOSING | 4 | 接続は閉じるハンドシェイクを行っており、まもなく閉じられます
### WebSocket閉じるフレーム状態コード

定数 | 対応値 | 説明
---|---|---
WEBSOCKET_CLOSE_NORMAL | 1000 | 正常に閉じられました。接続が完了しました。
WEBSOCKET_CLOSE_GOING_AWAY | 1001 | サーバーが切断されました。
WEBSOCKET_CLOSE_PROTOCOL_ERROR | 1002 | プロトコルエラーにより接続が中断されました。
WEBSOCKET_CLOSE_DATA_ERROR | 1003 | データエラーです。例えば、テキストデータが必要な場合にバイナリデータが受信された場合など。
WEBSOCKET_CLOSE_STATUS_ERROR | 1005 | 予期せぬステータスコードが受信されなかったことを表します。
WEBSOCKET_CLOSE_ABNORMAL | 1006 | 閉じるフレームが送信されませんでした。
WEBSOCKET_CLOSE_MESSAGE_ERROR | 1007 | 形式に合わないデータが受信されたため、接続が切断されました（例: テキストメッセージにUTF-8以外のデータが含まれている場合）。
WEBSOCKET_CLOSE_POLICY_ERROR | 1008 | 約束されたデータが受信されなかったため、接続が切断されました。これは一般的な状態コードであり、1003および1009の状態コードが適用されない場合に使用されます。
WEBSOCKET_CLOSE_MESSAGE_TOO_BIG | 1009 | 過大なデータフレームが受信されたため、接続が切断されました。
WEBSOCKET_CLOSE_EXTENSION_MISSING | 1010 | クライアントが1つ以上の拡張機能の取り決めを期待していますが、サーバーが処理しなかったため、クライアントは接続を切断しました。
WEBSOCKET_CLOSE_SERVER_ERROR | 1011 | クライアントが予期しない状況に遭遇し、リクエストを完了できなかったため、サーバーが接続を切断しました。
WEBSOCKET_CLOSE_TLS | 1015 | 予約済み。接続がTLSハンドシェイクを完了できなかったために閉じられました（例: サーバー証明書を検証できない場合）。
## オプション

?> `Swoole\WebSocket\Server`は`Server`のサブクラスであり、[Swoole\WebSocker\Server::set()](/server/methods?id=set)メソッドを使用して、構成オプションを渡して特定のパラメーターを設定できます。
### websocket_subprotocol

?> **Set the `WebSocket` subprotocol.**

?> After setting, the `HTTP` header of the handshake response will include `Sec-WebSocket-Protocol: {$websocket_subprotocol}`. Please refer to the relevant `RFC` document of the WebSocket protocol for specific usage.

```php
$server->set([
    'websocket_subprotocol' => 'chat',
]);
```
### open_websocket_close_frame

?> **`WebSocket`プロトコルでクローズフレーム（`opcode`が`0x08`のフレーム）を`onMessage`コールバックで受け取るようにするかどうかを設定します。デフォルトは`false`です。**

?> この設定を有効にすると、`Swoole\WebSocket\Server`の`onMessage`コールバックでクライアントまたはサーバーから送信されたクローズフレームを受信できます。開発者はこのフレームを自身で処理することができます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->set(array("open_websocket_close_frame" => true));
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x08) {
        echo "Close frame received: Code {$frame->code} Reason {$frame->reason}\n";
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {
});

$server->start();
```
### open_websocket_ping_frame

?> **`onMessage`コールバックでクライアントまたはサーバーからの`Ping`フレーム（`opcode`が`0x09`のフレーム）を受信するために`WebSocket`プロトコルの`Ping`フレームを有効にします。デフォルトは`false`です。**

?> 有効にすると、`Swoole\WebSocket\Server`の`onMessage`コールバックでクライアントまたはサーバーからの`Ping`フレームを受信でき、開発者はそれに対して自分で処理できます。

!> Swooleバージョン >= `v4.5.4` で利用可能

```php
$server->set([
    'open_websocket_ping_frame' => true,
]);
```

!> 値が`false`の場合、内部で自動的に`Pong`フレームが返信されますが、`true`に設定すると開発者自身が`Pong`フレームを返信する必要があります。

* **例**

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->set(array("open_websocket_ping_frame" => true));
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x09) {
        echo "Ping frame received: Code {$frame->opcode}\n";
        // Pong フレームを返信
        $pongFrame = new Swoole\WebSocket\Frame;
        $pongFrame->opcode = WEBSOCKET_OPCODE_PONG;
        $server->push($frame->fd, $pongFrame);
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {
});

$server->start();
```
### open_websocket_pong_frame

?> **`WebSocket`プロトコルの`Pong`フレーム（`opcode`が`0x0A`のフレーム）を`onMessage`コールバックで受信するかどうかを有効にします。デフォルトは`false`です。**

?> 有効にすると、`Swoole\WebSocket\Server`の`onMessage`コールバックでクライアントやサーバーから送信された`Pong`フレームを受信でき、開発者はそれを自由に処理できます。

!> Swooleのバージョン >= `v4.5.4` で利用可能

```php
$server->set([
    'open_websocket_pong_frame' => true,
]);
```

* **例**

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->set(array("open_websocket_pong_frame" => true));
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0xa) {
        echo "Pong frame received: Code {$frame->opcode}\n";
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {
});

$server->start();
```
### websocket_compression

**データの圧縮を有効にする**

`true`に設定すると、フレームを`zlib`で圧縮することが許可されます。具体的な圧縮が行われるかどうかは、クライアントが圧縮を処理できるかどうかに依存します（ハンドシェイク情報に基づき、`RFC-7692`を参照）。実際に特定のフレームを圧縮するには、`flags`パラメータの`SWOOLE_WEBSOCKET_FLAG_COMPRESS`を使用する必要があり、具体的な使用方法は[このセクションを参照してください](/websocket_server?id=websocket帧压缩-（rfc-7692）)。

Swooleバージョン >= `v4.4.12` で利用可能
## その他

!> 関連するサンプルコードは、[WebSocket ユニットテスト](https://github.com/swoole/swoole-src/tree/master/tests/swoole_websocket_server) で見つけることができます。
### WebSocket Frame Compression (RFC-7692)

?> First, you need to configure `'websocket_compression' => true` to enable compression (when `WebSocket` handshake, it will exchange compression support information with the peer), then you can use the `flag SWOOLE_WEBSOCKET_FLAG_COMPRESS` to compress a specific frame.
#### サンプル

* **サーバーサイド**

```php
use Swoole\WebSocket\Frame;
use Swoole\WebSocket\Server;

$server = new Server('127.0.0.1', 9501);
$server->set(['websocket_compression' => true]);
$server->on('message', function (Server $server, Frame $frame) {
    $server->push(
        $frame->fd,
        'Hello Swoole',
        SWOOLE_WEBSOCKET_OPCODE_TEXT,
        SWOOLE_WEBSOCKET_FLAG_FIN | SWOOLE_WEBSOCKET_FLAG_COMPRESS
    );
    // $server->push($frame->fd, $frame); // または、サーバーはクライアントのフレームオブジェクトをそのまま転送できます
});
$server->start();
```

* **クライアントサイド**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9501);
    $cli->set(['websocket_compression' => true]);
    $cli->upgrade('/');
    $cli->push(
        'Hello Swoole',
        SWOOLE_WEBSOCKET_OPCODE_TEXT,
        SWOOLE_WEBSOCKET_FLAG_FIN | SWOOLE_WEBSOCKET_FLAG_COMPRESS
    );
});
```
### Pingフレームの送信

WebSocketは長期間接続を維持するため、一定時間通信がないと接続が切断される可能性があります。そのため、心拍メカニズムが必要です。WebSocketプロトコルにはPingとPongの2つのフレームが含まれ、長期間接続を維持するために定期的にPingフレームを送信できます。
#### サンプル

* **サーバー**

```php
use Swoole\WebSocket\Frame;
use Swoole\WebSocket\Server;

$server = new Server('127.0.0.1', 9501);
$server->on('message', function (Server $server, Frame $frame) {
    $pingFrame = new Frame;
    $pingFrame->opcode = WEBSOCKET_OPCODE_PING;
    $server->push($frame->fd, $pingFrame);
});
$server->start();
```

* **クライアント**

```php
use Swoole\WebSocket\Frame;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9501);
    $cli->upgrade('/');
    $pingFrame = new Frame;
    $pingFrame->opcode = WEBSOCKET_OPCODE_PING;
    // PINGを送信
    $cli->push($pingFrame);
    
    // PONGを受信
    $pongFrame = $cli->recv();
    var_dump($pongFrame->opcode === WEBSOCKET_OPCODE_PONG);
});
```
