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

* **클라이언트**

  * `Chrome/Firefox/`고버전`IE/Safari`등 브라우저에는`JS`언어의`WebSocket`클라이언트가 내장되어 있습니다.
  * 웨이신 미니프로그램 개발 프레임워크에는 내장된`WebSocket`클라이언트가 있습니다.
  * [비동기 IO](/learn?id=동기io비동기io)의`PHP`프로그램에서는 [Swoole\Coroutine\Http](/coroutine_client/http_client)를`WebSocket`클라이언트로 사용할 수 있습니다.
  * `Apache/PHP-FPM`또는 다른 동기 차단`PHP`프로그램에서는`swoole/framework`가 제공하는 [동기 WebSocket 클라이언트](https://github.com/matyhtf/framework/blob/master/libs/Swoole/Client/WebSocket.php)를 사용할 수 있습니다.
  * `WebSocket`클라이언트가 아닌 경우`WebSocket`서버와 통신할 수 없습니다.

* **WebSocket 클라이언트인지 확인하는 방법**

?> [다음 예제](/server/methods?id=getclientinfo)를 사용하여 연결 정보를 가져와 배열이 반환되며, 이 배열에는 [websocket_status](/websocket_server?id=연결_상태)가 포함되어 있습니다. 이 상태를 기반으로`WebSocket`클라이언트인지 여부를 판단할 수 있습니다.
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    $client = $server->getClientInfo($frame->fd);
    // 또는 $client = $server->connection_info($frame->fd);
    if (isset($client['websocket_status'])) {
        echo "웹소켓 연결임";
    } else {
        echo "웹소켓 연결 아님";
    }
});
```
## 사건

?> `WebSocket` 서버는 [Swoole\Server](/server/methods) 및 [Swoole\Http\Server](/http_server) 기본 클래스의 콜백 함수를받을 수 있으며, 추가로`4`개의 콜백 함수가 설정되어 있습니다. 다음 중에서:

* `onMessage` 콜백 함수는 필수입니다.
* `onOpen`, `onHandShake` 및 `onBeforeHandShakeResponse` (Swoole5에서이 이벤트 제공) 콜백 함수는 선택 사항입니다.
### onBeforeHandshakeResponse

!> Swoole 버전 >= `v5.0.0`에서 사용 가능합니다.

?> **`WebSocket` 연결을 설정하기 전에 발생합니다. 사용자 정의 핸드셰이크 처리 과정이 필요하지 않지만 응답 헤더에 몇 가지 `http header` 정보를 설정하고 싶다면 이 이벤트를 호출할 수 있습니다.**

```php
onBeforeHandshakeResponse(Swoole\Http\Request $request, Swoole\Http\Response $response);
```
### onHandShake

?> **`WebSocket` 연결이 설정된 후에 핸드셰이크를 진행합니다. `WebSocket` 서버는 핸드셰이크 프로세스를 자동으로 수행하며, 사용자가 직접 핸드셰이크를 처리하려면 `onHandShake` 이벤트 콜백 함수를 설정할 수 있습니다.**

```php
onHandShake(Swoole\Http\Request $request, Swoole\Http\Response $response);
```

* **팁**

  * `onHandShake` 이벤트 콜백은 선택 사항입니다.
  * `onHandShake` 콜백 함수를 설정하면 더 이상 `onOpen` 이벤트가 발생하지 않으며, 응용 프로그램 코드에서 직접 처리해야 합니다. 필요한 경우 `$server->defer`를 사용하여 `onOpen` 로직을 호출할 수 있습니다.
  * `onHandShake` 메서드에서는 반드시 [response->status()](/http_server?id=status)를 호출하여 상태 코드를 `101`로 설정하고 [response->end()](/http_server?id=end)를 호출하여 응답해야 합니다. 그렇지 않으면 핸드셰이크가 실패합니다.
  * 내장된 핸드셰이크 프로토콜은 `Sec-WebSocket-Version: 13`입니다. 낮은 버전 브라우저의 경우 직접 핸드셰이크를 구현해야 합니다.

* **주의**

!> 만약 핸드셰이크를 직접 처리해야 하는 경우에만 이 콜백 함수를 설정하세요. "사용자 정의" 핸드셰이크 프로세스가 필요하지 않다면 이 콜백을 설정하지 마세요. 아래는 "사용자 정의" 핸드셰이크 이벤트 콜백 함수가 가져야 하는 내용입니다:

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // print_r( $request->header );
    // if (some custom conditions are not met, return end output, return false, handshake fails) {
    //    $response->end();
    //     return false;
    // }

    // WebSocket handshake connection algorithm verification
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

!> `onHandShake` 콜백 함수를 설정하면 더 이상 `onOpen` 이벤트가 발생하지 않으며, 응용 프로그램 코드에서 직접 처리해야 합니다. 필요한 경우 `$server->defer`를 사용하여 `onOpen` 로직을 호출할 수 있습니다.

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // 핸드셰이크 내용 생략
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

?> **웹소켓 클라이언트와 서버가 연결되고 핸드셰이크를 완료한 후에는이 함수가 콜백됩니다.**

```php
onOpen(Swoole\WebSocket\Server $server, Swoole\Http\Request $request);
```

* **팁**

    * `$request`는 클라이언트가 보낸 핸드셰이크 요청 정보를 포함하는 [HTTP](/http_server?id=httprequest) 요청 객체입니다.
    * `onOpen` 이벤트 함수에서는 [push](/websocket_server?id=push)를 사용하여 클라이언트에 데이터를 전송하거나 [close](/server/methods?id=close)를 호출하여 연결을 닫을 수 있습니다.
    * `onOpen` 이벤트 콜백은 선택 사항입니다.
### onMessage

?> **클라이언트로부터 데이터 프레임을 받았을 때 이 함수가 콜백됩니다.**

```php
onMessage(Swoole\WebSocket\Server $server, Swoole\WebSocket\Frame $frame)
```

* **팁**

  * `$frame`은 클라이언트가 보낸 데이터 프레임 정보를 포함하는 [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) 객체입니다.
  * `onMessage` 콜백은 설정되어 있어야 서버가 시작될 수 있습니다.
  * 클라이언트가 `ping` 프레임을 보내면 `onMessage`가 트리거되지 않으며, 하위 수준에서 자동적으로 `pong` 패킷을 응답하거나 수동으로 처리할 수 있도록 [open_websocket_ping_frame](/websocket_server?id=open_websocket_ping_frame) 매개변수를 설정할 수 있습니다.

!> `$frame->data`가 텍스트 유형이라면, 인코딩 형식은 꼭 `UTF-8`이어야 합니다. 이는 `WebSocket` 프로토콜에서 규정하는 것입니다.
### onRequest

?> `Swoole\WebSocket\Server`는 [Swoole\Http\Server](/http_server)을 상속했으므로 `Http\Server`가 제공하는 모든 `API` 및 구성 항목을 사용할 수 있습니다. [Swoole\Http\Server](/http_server) 섹션을 참조하십시오.

* [onRequest](/http_server?id=on) 콜백이 설정되어 있으면 `WebSocket\Server`는 동시에 `HTTP` 서버로 사용할 수 있습니다.
* [onRequest](/http_server?id=on) 콜백이 설정되어 있지 않으면 `WebSocket\Server`가 `HTTP` 요청을 받으면 `HTTP 400` 오류 페이지를 반환합니다.
* `HTTP`를 통해 모든 `WebSocket` 푸시를 트리거하려면 범위 문제에 유의해야 합니다. 절차지향으로 사용하려면 `global`을 사용하여 `Swoole\WebSocket\Server`을 참조하고, 객체지향으로 사용하려면 `Swoole\WebSocket\Server`를 멤버 속성으로 설정할 수 있습니다.
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
    global $server;//외부의 서버를 호출
    // $server->connections 모든 웹소켓 연결 사용자의 fd를 반복하고 모든 사용자에게 푸시
    foreach ($server->connections as $fd) {
        // 올바른 웹소켓 연결인지 먼저 확인해야 push 실패 가능성을 줄일 수 있음
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
            // HTTP 요청을 수신하여 get으로 message 매개변수의 값을 가져와 사용자에게 푸시합니다
            // $this->server->connections를 사용하여 모든 WebSocket 연결 사용자의 fd를 반복하고 모든 사용자에게 푸시합니다
            foreach ($this->server->connections as $fd) {
                // 올바른 WebSocket 연결인지 먼저 확인해야 합니다. 그렇지 않으면 푸시가 실패할 수 있습니다
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

?> **웹 소켓이 아닌 연결이 닫힐 때에만이 이벤트가 발생합니다.**

!> Swoole 버전 >= `v4.7.0`에서 사용 가능

```php
onDisconnect(Swoole\WebSocket\Server $server, int $fd)
```

!> `onDisconnect` 이벤트 콜백을 설정한 경우, 비 웹 소켓 요청이거나 [onRequest](/websocket_server?id=onrequest)에서 `$response->close()` 메서드를 호출한 경우에만 `onDisconnect` 콜백이 발생합니다. 그리고 [onRequest](/websocket_server?id=onrequest) 이벤트에서 정상적으로 종료된 경우 `onClose` 또는 `onDisconnect` 이벤트가 호출되지 않습니다.
## 방법

`Swoole\WebSocket\Server`는 [Swoole\Server](/server/methods)의 하위 클래스이므로 `Server`의 모든 메서드를 호출할 수 있습니다.

`WebSocket` 서버에서 클라이언트로 데이터를 보내려면 `Swoole\WebSocket\Server::push` 메서드를 사용해야 합니다. 이 메서드는 `WebSocket` 프로토콜을 사용하여 데이터를 패킹합니다. 반면 [Swoole\Server->send()](/server/methods?id=send) 메서드는 기본 `TCP` 전송 인터페이스입니다.

[Swoole\WebSocket\Server->disconnect()](/websocket_server?id=disconnect) 메서드를 사용하면 서버에서 `WebSocket` 연결을 수동으로 닫을 수 있습니다. [닫힌 상태 코드](/websocket_server?id=websocket-close-frames) (WebSocket 프로토콜에 따라 사용 가능한 상태 코드는 10진수 정수 중 하나이며 `1000` 또는 `4000-4999` 중 하나의 값을 취할 수 있음) 및 닫힌 이유(UTF-8로 인코딩되고 바이트 길이가 `125`를 초과하지 않는 문자열)를 지정할 수 있습니다. 상태 코드가 지정되지 않은 경우 `1000`이고 닫힌 이유는 비어 있습니다.
### push

?> **`WebSocket` 클라이언트 연결로 데이터를 푸시하며, 최대 길이는 `2M`여야 합니다.**

```php
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool

// v4.4.12 버전에서는 flags 매개변수로 변경됨
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): bool
```

* **파라미터** 

  * **`int $fd`**

    * **기능**：클라이언트 연결의 `ID` 【지정된 `$fd`가 `TCP` 연결이 WebSocket 클라이언트가 아닌 경우 전송에 실패함】
    * **기본값**：없음
    * **다른 값**：없음

  * **`Swoole\WebSocket\Frame|string $data`**

    * **기능**：전송할 데이터 내용
    * **기본값**：없음
    * **다른 값**：없음

  !> Swoole 버전 >= v4.2.0 인 경우, 전달된 `$data`가 [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) 객체인 경우 나머지 매개변수는 무시됨

  * **`int $opcode`**

    * **기능**：데이터 내용을 전송하는 형식을 지정함 【기본값은 텍스트. 이진 콘텐츠를 전송하려면 `$opcode` 매개변수를 `WEBSOCKET_OPCODE_BINARY`로 설정해야 합니다】
    * **기본값**：`WEBSOCKET_OPCODE_TEXT`
    * **다른 값**：`WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **기능**：전송 완료 여부
    * **기본값**：`true`
    * **다른 값**：`false`

* **반환값**

  * 작업이 성공하면`true`를 반환하고, 실패하면 `false`를 반환함

!> `v4.4.12` 버전부터 `finish` 매개변수(`bool`형)가 `flags` 매개변수(`int`형)로 변경되어 `WebSocket` 압축을 지원함, `finish`에 해당하는 `SWOOLE_WEBSOCKET_FLAG_FIN` 값은 `1`이며, 원래 `bool`형 값은 암시적으로 `int`형으로 변환됨. 이 변경 사항은 하위 호환성을 유지합니다. 또한 압축 `flag`은 `SWOOLE_WEBSOCKET_FLAG_COMPRESS`입니다.

!> [BASE 모드](/learn?id=base모드의_제한사항:) 는 다른 프로세스로의 `push` 데이터 전송을 지원하지 않음.
### 존재

?> **`WebSocket` 클라이언트가 존재하고 `Active` 상태인지 확인합니다.**

!> `v4.3.0` 이후에는 이 `API`는 연결의 존재 여부만 확인하는 데 사용되며, `WebSocket` 연결인지 확인하려면 `isEstablished`를 사용하십시오.

```php
Swoole\WebSocket\Server->exist(int $fd): bool
```

* **반환 값**

  * 연결이 존재하고 `WebSocket` 핸드셰이킹이 완료된 경우 `true`를 반환합니다.
  * 연결이 존재하지 않거나 핸드셰이킹이 아직 완료되지 않았을 경우 `false`를 반환합니다.
### 팩

?> **웹소켓 메시지를 패킹합니다.**

```php
Swoole\WebSocket\Server::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true, bool $mask = false): string

// v4.4.12 버전에서는 flags 매개변수로 변경
Swoole\WebSocket\Server::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): string

Swoole\WebSocket\Frame::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): string
```

* **매개변수** 

  * **`Swoole\WebSocket\Frame|string $data $data`**

    * **기능**：메시지 내용
    * **기본값**：없음
    * **기타 값**：없음

  * **`int $opcode`**

    * **기능**：데이터 형식 지정 【기본적으로 텍스트입니다. 이진 콘텐츠를 보내려면 `$opcode` 매개변수를 `WEBSOCKET_OPCODE_BINARY`로 설정해야 합니다.】
    * **기본값**：`WEBSOCKET_OPCODE_TEXT`
    * **기타 값**：`WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **기능**：프레임이 완료되었는지
    * **기본값**：없음
    * **기타 값**：없음

    !> `v4.4.12` 버전부터 `finish` 매개변수(`bool` 형식)가 `flags` 매개변수(`int` 형식)로 변경되어 `WebSocket` 압축을 지원합니다. `finish`은 `SWOOLE_WEBSOCKET_FLAG_FIN` 값을 `1`로 대응하며, 원래의 `bool` 값은 암시적으로 `int` 값으로 변환됩니다. 이 변경으로 하위 호환성에 영향을 미치지 않습니다.

  * **`bool $mask`**

    * **기능**：마스크 설정 여부【`v4.4.12`에서는 이 매개변수가 제거되었습니다.】
    * **기본값**：없음
    * **기타 값**：없음

* **반환 값**

  * 패킹된 `WebSocket` 데이터 패킷을 반환하며, `Swoole\Server` 기본 클래스의 [send()](/server/methods?id=send)를 사용하여 상대에게 전송할 수 있습니다.

* **예제**

```php
$ws = new Swoole\Server('127.0.0.1', 9501 , SWOOLE_BASE);

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

?> **WebSocket 데이터 프레임 해독.**

```php
Swoole\WebSocket\Server::unpack(string $data): Swoole\WebSocket\Frame|false;
```

* **매개변수** 

  * **`string $data`**

    * **기능** : 메시지 내용
    * **기본값** : 없음
    * **다른 값** : 없음

* **반환 값**

  * 해석에 실패하면 `false`를 반환하고, 성공하면 [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) 객체를 반환합니다.
### 연결 끊기

?> **`WebSocket` 클라이언트에게 종료 프레임을 보내고 해당 연결을 닫습니다.**

!> 작동이 가능한 Swoole 버전 >= `v4.0.3`

```php
Swoole\WebSocket\Server->disconnect(int $fd, int $code = SWOOLE_WEBSOCKET_CLOSE_NORMAL, string $reason = ''): bool
```

* **매개변수** 

  * **`int $fd`**

    * **기능**：클라이언트 연결의 `ID`【지정된 `$fd`가 해당하는 TCP 연결이 WebSocket 클라이언트가 아닌 경우 실패합니다】
    * **기본값**：없음
    * **다른 값**：없음

  * **`int $code`**

    * **기능**：연결을 끊는 상태 코드【`RFC6455`에 따라 애플리케이션 종료 상태 코드는 `1000` 또는 `4000-4999` 사이여야 합니다】
    * **기본값**：`SWOOLE_WEBSOCKET_CLOSE_NORMAL`
    * **다른 값**：없음

  * **`string $reason`**

    * **기능**：연결을 끊는 이유【`utf-8` 형식의 문자열이며 길이가 `125`바이트를 초과할 수 없습니다】
    * **기본값**：없음
    * **다른 값**：없음

* **반환값**

  * 성공적으로 전송되면 `true`를 반환하고, 전송에 실패하거나 상태 코드가 잘못된 경우 `false`를 반환합니다.
### isEstablished

?> **`WebSocket` 클라이언트 연결이 유효한지 확인합니다.**

?> 이 함수는 `exist` 메소드와 다릅니다. `exist` 메소드는 `TCP` 연결 여부만 확인하고 완료된 `WebSocket` 클라이언트인지 판단할 수 없습니다.

```php
Swoole\WebSocket\Server->isEstablished(int $fd): bool
```

* **매개변수** 

  * **`int $fd`**

    * **기능**: 클라이언트 연결의 `ID`【지정된 `$fd`가 `TCP` 연결이 **WebSocket** 클라이언트가 아닌 경우 실패합니다】
    * **기본값**: 없음
    * **기타 값**: 없음

* **반환 값**

  * 유효한 연결인 경우 `true`를 반환하고, 그렇지 않으면 `false`를 반환합니다.
## 웹소켓 데이터 프레임 프레임 클래스
### Swoole\WebSocket\Frame

`v4.2.0` 버전에서는 서버 및 클라이언트가 [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) 객체를 보낼 수 있게되었습니다.  
`v4.4.12` 버전에서는 `flags` 속성을 추가하여 `WebSocket` 압축 프레임을 지원하고 새로운 하위 클래스 [Swoole\WebSocket\CloseFrame](/websocket_server?id=swoolewebsocketcloseframe)을 추가했습니다.

일반적인 `frame` 객체에는 다음과 같은 속성이 있습니다.

상수 | 설명
---|---
fd | 클라이언트의 `소켓 ID`로, 데이터를 보내기 위해서는 `$server->push`를 사용해야합니다.
data | 데이터 내용으로, 텍스트 내용 또는 이진 데이터가 될 수 있으며 `opcode`의 값으로 구분할 수 있습니다.
opcode | `WebSocket`의 [데이터 프레임 유형](/websocket_server?id=데이터프레임유형)을 나타내며, `WebSocket` 프로토콜 표준 문서를 참조할 수 있습니다.
finish | 데이터 프레임이 완전한지 여부를 나타내며, 하나의 `WebSocket` 요청은 여러 데이터 프레임으로 분할될 수 있습니다. (하위 수준에서 데이터 프레임을 자동으로 병합하도록 구현되었으므로 이제 데이터 프레임이 불완전하다고 걱정할 필요가 없습니다.)

이 클래스에는 [Swoole\WebSocket\Frame::pack()](/websocket_server?id=pack) 및 [Swoole\WebSocket\Frame::unpack()](/websocket_server?id=unpack)가 내장되어 있으며, `websocket` 메시지를 패킹하고 언패킹하기 위해 사용됩니다. 매개변수 설명은 `Swoole\WebSocket\Server::pack()` 및 `Swoole\WebSocket\Server::unpack()`과 동일합니다.
### Swoole\WebSocket\CloseFrame

일반적인 `닫기 프레임 close frame` 객체는 다음과 같은 속성을 갖습니다.

상수 | 설명
---|---
opcode | `WebSocket`의 [데이터 프레임 유형](/websocket_server?id=데이터프레임타입)은 `WebSocket` 프로토콜 표준 문서를 참조할 수 있습니다.
code | `WebSocket`의 [닫기 프레임 상태 코드](/websocket_server?id=WebSocket닫기상태코드)는 [websocket 프로토콜에서 정의된 오류 코드](https://developer.mozilla.org/zh-CN/docs/Web/API/CloseEvent)를 참조할 수 있습니다.
reason | 닫히는 이유, 명시적으로 지정되지 않으면 비어 있음

서버가 `close frame`을 수신해야 하는 경우 `$server->set`을 통해 [open_websocket_close_frame](/websocket_server?id=open_websocket_close_frame) 매개변수를 활성화해야 합니다.
## 상수
### 데이터 프레임 유형

상수 | 값 | 설명
---|---|---
WEBSOCKET_OPCODE_TEXT | 0x1 | UTF-8 텍스트 문자 데이터
WEBSOCKET_OPCODE_BINARY | 0x2 | 이진 데이터
WEBSOCKET_OPCODE_CLOSE | 0x8 | 닫기 프레임 유형 데이터
WEBSOCKET_OPCODE_PING | 0x9 | 핑 타입 데이터
WEBSOCKET_OPCODE_PONG | 0xa | 퐁 타입 데이터
### 연결 상태

상수 | 값 | 설명
---|---|---
WEBSOCKET_STATUS_CONNECTION | 1 | 연결 대기 중
WEBSOCKET_STATUS_HANDSHAKE | 2 | 핸드셰이크 진행 중
WEBSOCKET_STATUS_ACTIVE | 3 | 핸드셰이크 완료, 브라우저가 데이터 프레임을 보낼 때 대기 중
WEBSOCKET_STATUS_CLOSING | 4 | 연결 종료 핸드셰이크 진행 중, 곧 종료됨
### WebSocket 클로증 프레임 상태 코드

안수 | 대응 값 | 설명
---|---|---
WEBSOCKET_CLOSE_NORMAL | 1000 | 정상 종료, 연결이 완료되었습니다.
WEBSOCKET_CLOSE_GOING_AWAY | 1001 | 서버에서 연결이 끊김
WEBSOCKET_CLOSE_PROTOCOL_ERROR | 1002 | 프로토콜 오류로 연결이 중단됨
WEBSOCKET_CLOSE_DATA_ERROR | 1003 | 데이터 오류, 예를 들어 텍스트 데이터가 필요한데 이진 데이터가 수신된 경우
WEBSOCKET_CLOSE_STATUS_ERROR | 1005 | 예상 상태 코드를 수신하지 못한 경우
WEBSOCKET_CLOSE_ABNORMAL | 1006 | 종료 프레임을 보내지 않은 경우
WEBSOCKET_CLOSE_MESSAGE_ERROR | 1007 | 형식에 맞지 않는 데이터를 수신하여 연결이 끊김 (예: 텍스트 메시지에 UTF-8이 아닌 데이터 포함)
WEBSOCKET_CLOSE_POLICY_ERROR | 1008 | 협약에 맞지 않는 데이터를 수신하여 연결이 끊김. 이는 1003 및 1009 상태 코드를 사용하기에 적합하지 않은 일반 상태 코드입니다.
WEBSOCKET_CLOSE_MESSAGE_TOO_BIG | 1009 | 너무 큰 데이터 프레임을 수신하여 연결이 끊김
WEBSOCKET_CLOSE_EXTENSION_MISSING | 1010 | 클라이언트가 하나 이상의 확장을 서버와 합의하기를 기대했지만 서버가 처리하지 않아 클라이언트가 연결을 끊은 경우
WEBSOCKET_CLOSE_SERVER_ERROR | 1011 | 클라이언트가 예상치 못한 상황으로 인해 요청을 완료할 수 없어 서버가 연결을 끊은 경우
WEBSOCKET_CLOSE_TLS | 1015 | 예약됨. TLS 핸드쉐이크를 완료할 수 없어 연결이 닫힌 경우 (예: 서버 인증서를 확인할 수 없음).
## 옵션

?> `Swoole\WebSocket\Server`는 `Server`의 자식 클래스이며 [Swoole\WebSocker\Server::set()](/server/methods?id=set) 메서드를 사용하여 구성 옵션을 전달하여 특정 매개변수를 설정할 수 있습니다.
### websocket_subprotocol

?> **`WebSocket` 하위 프로토콜을 설정합니다.**

?> 설정하면 핸드셰이크 응답의 `HTTP` 헤더에 `Sec-WebSocket-Protocol: {$websocket_subprotocol}`이 추가됩니다. 자세한 사용 방법은 `WebSocket` 프로토콜 관련 `RFC` 문서를 참조하십시오.

```php
$server->set([
    'websocket_subprotocol' => 'chat',
]);
```
### open_websocket_close_frame

?> **`WebSocket` 프로토콜에서 클로즈 프레임(`opcode`이 `0x08`인 프레임)을 `onMessage` 콜백에서 수신하도록 활성화합니다. 기본값은 `false`입니다.**

?> 활성화하면 `Swoole\WebSocket\Server`의 `onMessage` 콜백에서 클라이언트 또는 서버가 보낸 클로즈 프레임을 수신할 수 있으며, 개발자가 직접 처리할 수 있습니다.

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

?> **'WebSocket' 프로토콜에서 'Ping' 프레임 (`opcode`가 `0x09` 인 프레임)을 'onMessage' 콜백에서 수신하도록하는 옵션입니다. 기본값은 `false`입니다.**

?> 활성화하면 'Swoole\WebSocket\Server'의 'onMessage' 콜백에서 클라이언트 또는 서버에서 보낸 'Ping' 프레임을 수신할 수 있으며, 개발자가 직접 처리할 수 있습니다.

!> Swoole 버전 >= `v4.5.4`에서 사용 가능합니다.

```php
$server->set([
    'open_websocket_ping_frame' => true,
]);
```

!> 값이 `false`로 설정된 경우에는 하위 레벨에서 자동으로 'Pong' 프레임을 응답하지만, `true`로 설정한 후에는 개발자가 직접 'Pong' 프레임을 응답해야 합니다.

* **예시**

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->set(array("open_websocket_ping_frame" => true));
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x09) {
        echo "Ping frame received: Code {$frame->opcode}\n";
        // 'Pong' 프레임 응답
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

?> **`WebSocket` 프로토콜에서 `Pong` 프레임(`opcode`가 `0x0A`인 프레임)을 `onMessage` 콜백에서 수신하도록 활성화합니다. 기본값은 `false`입니다.**

?> 활성화하면 `Swoole\WebSocket\Server`의 `onMessage` 콜백에서 클라이언트 또는 서버에서 보낸 `Pong` 프레임을 받을 수 있으며, 개발자는 해당 프레임을 자체적으로 처리할 수 있습니다.

!> Swoole 버전 >= `v4.5.4`에서 사용 가능

```php
$server->set([
    'open_websocket_pong_frame' => true,
]);
```

* **예제**

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
### 웹소켓 압축

?> **데이터 압축 활성화**

?> `true`로 설정하면 프레임을 `zlib`로 압축할 수 있습니다. 압축이 가능한지 여부는 클라이언트가 압축을 처리할 수 있는지에 따라 달라집니다(핸드셰이크 정보에 따라 결정됨, `RFC-7692` 참조). 특정 프레임을 실제로 압축하려면 `flags` 매개변수 `SWOOLE_WEBSOCKET_FLAG_COMPRESS`와 함께 사용해야 합니다. 구체적인 사용 방법은 [이 부분](/websocket_server?id=websocket帧压缩-（rfc-7692）)을 참조하십시오.

!> Swoole 버전 >= `v4.4.12`에서 사용 가능
## 기타

!> 관련 예제 코드는 [WebSocket 단위 테스트](https://github.com/swoole/swoole-src/tree/master/tests/swoole_websocket_server)에서 찾을 수 있습니다.
### WebSocket Frame Compression (RFC-7692)

?> 먼저 압축을 활성화하려면 `'websocket_compression' => true`을 구성해야 합니다. 이렇게 하면 (`WebSocket` 핸드 셰이크시 상대방과의 압축 지원 정보가 교환됩니다). 그런 다음 특정 프레임을 압축하려면 ` flag SWOOLE_WEBSOCKET_FLAG_COMPRESS`를 사용할 수 있습니다.
#### 예시

* **서버 측**

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
    // $server->push($frame->fd, $frame); // 또는 서버는 클라이언트의 프레임 개체를 그대로 전달할 수 있습니다.
});
$server->start();
```

* **클라이언트 측**

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
### Ping 프레임 보내기

?> 웹소켓은 장기 연결이기 때문에 일정 시간 동안 통신이 없으면 연결이 끊길 수 있습니다. 이때 하트비트 메커니즘이 필요하며, 웹소켓 프로토콜에는 Ping과 Pong 두 개의 프레임이 포함되어 있어서 장기 연결 유지를 위해 주기적으로 Ping 프레임을 전송할 수 있습니다.
#### 예시

* **서버 측**

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

* **클라이언트 측**

```php
use Swoole\WebSocket\Frame;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9501);
    $cli->upgrade('/');
    $pingFrame = new Frame;
    $pingFrame->opcode = WEBSOCKET_OPCODE_PING;
    // PING 보내기
    $cli->push($pingFrame);
    
    // PONG 받기
    $pongFrame = $cli->recv();
    var_dump($pongFrame->opcode === WEBSOCKET_OPCODE_PONG);
});
```
