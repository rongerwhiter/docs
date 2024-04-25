# WebSocket 서버

?> 완전히 코루틴화된 WebSocket 서버 구현은 [Coroutine\Http\Server](/coroutine/http_server)를 상속받았으며, `WebSocket` 프로토콜을 지원하는 기반을 제공합니다. 여기에서는 차이점에 대해서만 언급합니다.

!> 이 섹션은 v4.4.13 이후에 사용 가능합니다.
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
### 다중 전송 예시

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
## 처리 과정

* `$ws->upgrade()`：클라이언트에 `WebSocket` 핸드쉐이크 메시지를 보냅니다
* `while(true)` 루프를 통해 메시지 수신 및 송신 처리를 계속합니다
* `$ws->recv()`는 `WebSocket` 메시지 프레임을 수신합니다
* `$ws->push()`는 상대에게 데이터 프레임을 전송합니다
* `$ws->close()`는 연결을 닫습니다

!> `$ws`는 `Swoole\Http\Response` 개체이며, 각 메소드의 사용법은 아래를 참조하세요.
번역할 텍스트가 없습니다.
### upgrade()

`웹소켓 WebSocket` 핸드쉐이크 성공 메시지를 전송합니다.

!> 이 메소드는 [비동기 스타일](/http_server)의 서버에서 사용하지 마십시오.

```php
Swoole\Http\Response->upgrade(): bool
```
### recv()

`WebSocket` 메시지를 수신합니다.

!> 이 메서드는 [비동기 스타일](/http_server)의 서버에서 사용하지 마십시오. `recv` 메서드를 호출하면 현재 코루틴이 [일시 중지](/coroutine?id=코루틴 스케줄링)되어 데이터가 도착할 때까지 대기한 후 코루틴 실행을 다시 시작합니다.

```php
Swoole\Http\Response->recv(float $timeout = 0): Swoole\WebSocket\Frame | false | string
```

* **반환값**

  * 메시지를 성공적으로 받으면 `Swoole\WebSocket\Frame` 객체가 반환됩니다. 자세한 내용은 [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)을 참조하십시오.
  * 실패하면 `false`가 반환되며, 에러 코드를 얻으려면 [swoole_last_error()](/functions?id=swoole_last_error)를 사용하십시오.
  * 연결이 닫히면 빈 문자열이 반환됩니다.
  * 반환 값 처리에 대한 예시는 [브로드캐스트 예시](/coroutine/ws_server?id=브로드캐스트-예시)를 참조하십시오.
### push()

`WebSocket` 데이터 프레임을 보냅니다.

!> 이 메서드는 [비동기 스타일](/http_server)의 서버에서 사용하지 마십시오. 대량의 데이터 패킷을 전송할 때는 쓰기 가능한 상태를 계속 감시해야 하므로 여러 번의 [코루틴 전환](/coroutine?id=코루틴 스케줄링)이 발생할 수 있습니다.

```php
Swoole\Http\Response->push(string|object $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool
```

* **매개변수** 

  !> 만약 전달되는 `$data`가 [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) 객체이면 후속 매개변수는 무시되고 다양한 프레임 유형을 보낼 수 있습니다.

  * **`string|object $data`**

    * **기능**: 보낼 내용
    * **기본값**: 없음
    * **다른 값**: 없음

  * **`int $opcode`**

    * **기능**: 데이터 내용을 지정하는 형식을 지정합니다. [기본적으로 텍스트입니다. 이진 내용을 보내려면 `$opcode` 매개변수를 `WEBSOCKET_OPCODE_BINARY`로 설정해야 합니다.]
    * **기본값**: `WEBSOCKET_OPCODE_TEXT`
    * **다른 값**: `WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **기능**: 전송 완료 여부
    * **기본값**: `true`
    * **다른 값**: `false`
### close()

`WebSocket` 연결을 닫습니다.

!> 이 방법은 [비동기 스타일](/http_server)의 서버에서 사용하지 말아야 합니다. v4.4.15 이전 버전에서는 `Warning`이 잘못 발생할 수 있으니 무시하십시오.

```php
Swoole\Http\Response->close(): bool
```

이 메서드는 `TCP` 연결을 직접 종료하며 `Close` 프레임을 보내지 않습니다. 이는 `WebSocket\Server::disconnect()` 메서드와 다릅니다. 
연결을 닫기 전에 `push()` 메소드를 사용하여 `Close` 프레임을 보내어 클라이언트에 알릴 수 있습니다.

```php
$frame = new Swoole\WebSocket\CloseFrame;
$frame->reason = 'close';
$ws->push($frame);
$ws->close();
```
