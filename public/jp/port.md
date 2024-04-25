# 複数のポートのリスニング

`Swoole\Server`は複数のポートをリスニングすることができ、各ポートには異なるプロトコル処理方法を設定できます。たとえば、80番ポートではHTTPプロトコルを処理し、9507番ポートではTCPプロトコルを処理します。`SSL/TLS`転送暗号化も特定のポートにのみ有効にすることができます。

!> 例えば、メインサーバーがWebSocketまたはHTTPプロトコルの場合、新しくリスンするTCPポート（`listen`メソッドの戻り値である`Swoole\Server\Port`オブジェクト、以下略して port とします）は、デフォルトでメインServerのプロトコル設定を継承します。新しいプロトコルを有効にするには、`port`オブジェクトの`set`メソッドと`on`メソッドを個別に呼び出して新しいプロトコルを設定する必要があります。
## 新しいポートのリッスン

```php
//ポートオブジェクトを返す
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port2 = $server->listen("127.0.0.1", 9502, SWOOLE_SOCK_UDP);
$port3 = $server->listen("127.0.0.1", 9503, SWOOLE_SOCK_TCP | SWOOLE_SSL);
```
```php
//portオブジェクトのsetメソッドの呼び出し
$port1->set([
	'open_length_check' => true,
	'package_length_type' => 'N',
	'package_length_offset' => 0,
	'package_max_length' => 800000,
]);

$port3->set([
	'open_eof_split' => true,
	'package_eof' => "\r\n",
	'ssl_cert_file' => 'ssl.cert',
	'ssl_key_file' => 'ssl.key',
]);
```
## コールバック関数の設定

```php
//それぞれのポートにコールバック関数を設定します
$port1->on('connect', function ($serv, $fd){
    echo "Client:Connect.\n";
});

$port1->on('receive', function ($serv, $fd, $reactor_id, $data) {
    $serv->send($fd, 'Swoole: '.$data);
    $serv->close($fd);
});

$port1->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

$port2->on('packet', function ($serv, $data, $addr) {
    var_dump($data, $addr);
});
```

!> `Swoole\Server\Port`の詳細については[こちら](/server/server_port)をクリックしてください。
## Http/WebSocket

`Swoole\Http\Server`および`Swoole\WebSocket\Server`は、継承されたサブクラスで実装されているため、`Swoole\Server`インスタンスの`listen`メソッドを呼び出してHTTPサーバーまたはWebSocketサーバーを作成することはできません。

サーバーの主な機能が`RPC`であるが、簡単なWeb管理画面を提供したい場合、まず`HTTP/WebSocket`サーバーを作成し、その後ネイティブTCPポートで`listen`を行うことができます。
```php
$http_server = new Swoole\Http\Server('0.0.0.0',9998);
$http_server->set(['daemonize'=> false]);
$http_server->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

//多监听一个TCP端口，对外开启TCP服务，并设置TCP服务器的回调
$tcp_server = $http_server->listen('0.0.0.0', 9999, SWOOLE_SOCK_TCP);
//默认新监听的端口 9999 会继承主服务器的设置，也是 HTTP 协议
//需要调用 set 方法覆盖主服务器的设置
$tcp_server->set([]);
$tcp_server->on('receive', function ($server, $fd, $threadId, $data) {
    echo $data;
});

$http_server->start();
```

このコードを使用すると、外部にHTTPサービスを提供し、同時に外部にTCPサービスを提供するサーバーを構築できます。より具体的でエレガントなコードの組み合わせは、あなた自身が実装してください。
```php
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port1->set([
    'open_websocket_protocol' => true, // このポートでWebSocketプロトコルをサポートするように設定する
]);
```

```php
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port1->set([
    'open_http_protocol' => false, // このポートでHTTPプロトコル機能を無効にする
]);
```

同様に、`open_http_protocol`、`open_http2_protocol`、`open_mqtt_protocol` などのパラメータもあります。
## オプションパラメーター

* ポート`port`が`set`メソッドを呼ばれていない場合、プロトコル処理オプションのリッスンポートを設定せず、それはメインサーバーの関連設定を継承します
* メインサーバーが`HTTP/WebSocket`サーバーである場合、プロトコルパラメーターが設定されていない場合でも、リッスンされるポートは引き続き`HTTP`または`WebSocket`プロトコルに設定され、ポートに設定された[onReceive](/server/events?id=onreceive)コールバックを実行しません
* メインサーバーが`HTTP/WebSocket`サーバーである場合、リッスンポートが`set`で設定された構成パラメータを持つと、メインサーバーのプロトコル設定がクリアされます。リッスンポートは`TCP`プロトコルに変わります。ポートが引き続き`HTTP/WebSocket`プロトコルを使用する場合は、構成に`open_http_protocol => true`および`open_websocket_protocol => true`を追加する必要があります

**`port`に`set`メソッドで設定できるパラメーターは以下のとおりです：**

* ソケットパラメータ：`backlog`、`open_tcp_keepalive`、`open_tcp_nodelay`、`tcp_defer_accept`など
* プロトコル関連：`open_length_check`、`open_eof_check`、`package_length_type`など
* SSL証明書関連：`ssl_cert_file`、`ssl_key_file`など

詳細は[設定セクション](/server/setting)を参照してください
## Optional Callbacks

`port`未调用`on`方法，设置回调函数的监听端口，默认使用主服务器的回调函数，`port`可以通过`on`方法设置的回调有： 

I provided the translation above.
### TCPサーバー

* onConnect
* onClose
* onReceive
### UDP サーバー

* onPacket
* onReceive
### HTTPサーバー

* onRequest
### WebSocketサーバー

* onMessage
* onOpen
* onHandshake

!> 異なるポートをリッスンするコールバック関数でも、同じ `Worker` プロセス空間で実行されます
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9514, SWOOLE_BASE);

$tcp = $server->listen("0.0.0.0", 9515, SWOOLE_SOCK_TCP);
$tcp->set([]);

$server->on("open", function ($serv, $req) {
    echo "新しい WebSocket クライアント、fd={$req->fd}\n";
});

$server->on("message", function ($serv, $frame) {
    echo "{$frame->fd} から受信：{$frame->data}、opcode：{$frame->opcode}、fin：{$frame->finish}\n";
    $serv->push($frame->fd, "これはサーバーのOnMessageです");
});

$tcp->on('receive', function ($server, $fd, $reactor_id, $data) {
    // 9514ポートの接続をだけを網羅し、これは$tcpではなく$serverを使用しているため
    $websocket = $server->ports[0];
    foreach ($websocket->connections as $_fd) {
        var_dump($_fd);
        if ($server->exist($_fd)) {
            $server->push($_fd, "これはサーバーのonReceiveです");
        }
    }
    $server->send($fd, '受信：'.$data);
});

$server->start();
```
