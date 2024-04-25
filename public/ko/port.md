# 다중 포트 대기

`Swoole\Server`는 여러 포트를 대기할 수 있습니다. 각 포트는 다른 프로토콜 처리 방식을 설정할 수 있습니다. 예를 들어 80번 포트는 HTTP 프로토콜을 처리하고, 9507번 포트는 TCP 프로토콜을 처리할 수 있습니다. `SSL/TLS` 전송 암호화는 특정 포트에만 활성화할 수도 있습니다.

!> 예를 들어, 주 서버가 WebSocket 또는 HTTP 프로토콜을 사용하면, 새로운 TCP 포트(즉, `Swoole\Server\Port` 객체로 반환되는 [listen](/server/methods?id=listen)의 결과물, 이하 "포트"로 지칭함)는 기본적으로 주 서버의 프로토콜 설정을 상속받습니다. 새로운 프로토콜을 활성화하려면 `port` 객체의 `set` 및 `on` 메서드를 인두적으로 호출하여 새로운 프로토콜을 설정해야 합니다.
```php
// port 객체 반환
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port2 = $server->listen("127.0.0.1", 9502, SWOOLE_SOCK_UDP);
$port3 = $server->listen("127.0.0.1", 9503, SWOOLE_SOCK_TCP | SWOOLE_SSL);
```
```php
//port 객체의 set 메서드 호출
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
## 콜백 함수 설정

```php
// 각 포트의 콜백 함수 설정
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

!> `Swoole\Server\Port`의 자세한 설명은 [여기를](/server/server_port) 클릭하세요.
## Http/WebSocket

`Swoole\Http\Server`와 `Swoole\WebSocket\Server`는 서브 클래스를 사용하기 때문에 `Swoole\Server` 인스턴스의 `listen` 메서드를 호출하여 HTTP 또는 WebSocket 서버를 생성할 수 없습니다.

서버의 주요 기능이 `RPC`인데도 간단한 웹 관리 인터페이스를 제공하고 싶은 경우, `HTTP/WebSocket` 서버를 먼저 만들고 나서 원시 TCP 포트를 듣기 위해 `listen`을 수행할 수 있습니다.
### 예시

```php
$http_server = new Swoole\Http\Server('0.0.0.0',9998);
$http_server->set(['daemonize'=> false]);
$http_server->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

//더 많은 TCP 포트를 수신하여 외부 TCP 서비스를 시작하고 TCP 서버 콜백을 설정합니다
$tcp_server = $http_server->listen('0.0.0.0', 9999, SWOOLE_SOCK_TCP);
//기본적으로 새로운 수신 포트 9999는 주 서버 설정을 그대로 사용하지만, HTTP 프로토콜입니다.
//주 서버 설정을 덮어쓰기 위해서는 set 메소드를 호출해야 합니다.
$tcp_server->set([]);
$tcp_server->on('receive', function ($server, $fd, $threadId, $data) {
    echo $data;
});

$http_server->start();
```

이 코드를 통해 HTTP 서비스를 외부로 제공하면서 동시에 외부 TCP 서비스를 제공하는 서버를 구축할 수 있습니다. 더 구체적이고 우아한 코드 조합은 여러분이 직접 구현해야 합니다.
```php
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port1->set([
    'open_websocket_protocol' => true, // 이 포트에서 WebSocket 프로토콜을 지원하도록 설정
]);
```

```php
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port1->set([
    'open_http_protocol' => false, // 이 포트에서 HTTP 프로토콜 기능을 비활성화
]);
```

같은 방식으로 `open_http_protocol`, `open_http2_protocol`, `open_mqtt_protocol` 등의 매개변수도 있습니다.
## 선택적 매개변수

* `port` 매개변수를 `set` 메서드로 호출하지 않을 경우, 프로토콜 처리 옵션을 설정하지 않고 리슨 포트를 설정하면 주 서버의 관련 구성을 상속받게 됩니다.
* 주 서버가 `HTTP/WebSocket` 서버인 경우, 프로토콜 매개변수를 설정하지 않으면, 리스닝 포트는 여전히 `HTTP` 또는 `WebSocket` 프로토콜로 설정되며, 리스닝 포트에 대한 [onReceive](/server/events?id=onreceive) 콜백을 수행하지 않습니다.
* 주 서버가 `HTTP/WebSocket` 서버인 경우, 리스닝 포트에서 `set`를 사용하여 구성 매개변수를 설정하면, 주 서버의 프로토콜 설정이 지워집니다. 리스닝 포트는 `TCP` 프로토콜로 변경됩니다. 원하는 경우 리스닝 포트가 여전히 `HTTP/WebSocket` 프로토콜을 사용하려면 구성에 `open_http_protocol => true` 및 `open_websocket_protocol => true`를 추가해야 합니다.

**`port`에서 `set`으로 설정할 수 있는 매개변수는 다음과 같습니다:**

* 소켓 매개변수: `backlog`, `open_tcp_keepalive`, `open_tcp_nodelay`, `tcp_defer_accept` 등
* 프로토콜 관련: `open_length_check`, `open_eof_check`, `package_length_type` 등
* SSL 인증서 관련: `ssl_cert_file`, `ssl_key_file` 등

자세한 내용은 [설정 섹션](/server/setting)을 참조하세요.
## 선택 콜백

`port`가 `on` 메서드를 호출하지 않았을 때, 콜백 함수의 대기 포트로는 기본적으로 메인 서버의 콜백 함수가 사용됩니다. `port`에서 설정할 수 있는 `on` 메서드로 설정할 수 있는 콜백은 다음과 같습니다:
### TCP 서버

* onConnect
* onClose
* onReceive
### UDP 서버

* onPacket
* onReceive
### HTTP 서버

* onRequest
### 웹 소켓 서버

* onMessage
* onOpen
* onHandshake

!> 서로 다른 포트를 수신하는 콜백 함수는 여전히 동일한 `Worker` 프로세스 내에서 실행됩니다.
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9514, SWOOLE_BASE);

$tcp = $server->listen("0.0.0.0", 9515, SWOOLE_SOCK_TCP);
$tcp->set([]);

$server->on("open", function ($serv, $req) {
    echo "새로운 WebSocket 클라이언트, fd={$req->fd}\n";
});

$server->on("message", function ($serv, $frame) {
    echo "받은 데이터: {$frame->fd}:{$frame->data}, opcode:{$frame->opcode}, fin:{$frame->finish}\n";
    $serv->push($frame->fd, "서버에서 보낸 메시지입니다");
});

$tcp->on('receive', function ($server, $fd, $reactor_id, $data) {
    // 9514 포트의 연결만 순회하므로 $server를 사용하고 있으므로
    $websocket = $server->ports[0];
    foreach ($websocket->connections as $_fd) {
        var_dump($_fd);
        if ($server->exist($_fd)) {
            $server->push($_fd, "서버에서 받은 메시지입니다");
        }
    }
    $server->send($fd, '수신한 데이터: '.$data);
});

$server->start();
```
