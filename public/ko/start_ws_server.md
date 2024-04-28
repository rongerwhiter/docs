# 웹 소켓 서버

## 프로그램 코드

다음 코드를 websocketServer.php에 작성해 주세요.

```php
//WebSocket 서버 객체를 생성하고 0.0.0.0:9502 포트를 수신 대기합니다.
$ws = new Swoole\WebSocket\Server('0.0.0.0', 9502);

//WebSocket 연결이 열릴 때 이벤트를 수신 대기합니다.
$ws->on('Open', function ($ws, $request) {
    $ws->push($request->fd, "hello, welcome\n");
});

//WebSocket 메시지 이벤트를 수신 대기합니다.
$ws->on('Message', function ($ws, $frame) {
    echo "Message: {$frame->data}\n";
    $ws->push($frame->fd, "server: {$frame->data}");
});

//WebSocket 연결이 닫힐 때 이벤트를 수신 대기합니다.
$ws->on('Close', function ($ws, $fd) {
    echo "client-{$fd} is closed\n";
});

$ws->start();
```

* 클라이언트가 서버로 메시지를 보낼 때 서버는 `onMessage` 이벤트를 호출합니다.
* 서버는 `$server->push()`를 사용하여 특정 클라이언트($fd 식별자 사용)에게 메시지를 보낼 수 있습니다.

## 프로그램 실행

```shell
php websocketServer.php
```

Google Chrome 브라우저를 사용하여 테스트할 수 있습니다. JavaScript 코드는 다음과 같습니다.

```javascript
var wsServer = 'ws://127.0.0.1:9502';
var websocket = new WebSocket(wsServer);
websocket.onopen = function (evt) {
    console.log("Connected to WebSocket server.");
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
```

## 코메트 (Comet)

웹 소켓 서버는 웹 소켓 기능뿐만 아니라 HTTP 긴 폴링을 처리할 수도 있습니다. 단순히 [onRequest](/http_server?id=on) 이벤트 청취를 추가하면 코메트 방식의 HTTP 긴 폴링을 구현할 수 있습니다.

!> 자세한 사용법은 [Swoole\WebSocket](/websocket_server)를 참조하십시오.
