# TCP 서버

## 프로그램 코드

다음 코드를 tcpServer.php에 작성하십시오.

```php
//Server 객체를 생성하여 127.0.0.1:9501 포트를 수신합니다.
$server = new Swoole\Server('127.0.0.1', 9501);

//연결 수신 이벤트를 듣습니다.
$server->on('Connect', function ($server, $fd) {
    echo "Client: Connect.\n";
});

//데이터 수신 이벤트를 듣습니다.
$server->on('Receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, "Server: {$data}");
});

//연결 닫힘 이벤트를 듣습니다.
$server->on('Close', function ($server, $fd) {
    echo "Client: Close.\n";
});

//서버를 시작합니다.
$server->start();
```

이렇게 하면 로컬 호스트 9501 포트를 수신하는 `TCP` 서버가 생성됩니다. 이 서버의 논리는 매우 간단합니다. 클라이언트`Socket`이 `hello` 문자열을 네트워크를 통해 보낼 때, 서버는 `Server: hello` 문자열을 다시 보냅니다.

`Server`는 비동기 서버이므로 이벤트를 청취하여 프로그램을 작성합니다. 해당하는 이벤트가 발생하면 밑바닥에서 특정 함수를 호출합니다. 예를 들어 새로운 `TCP` 연결이 들어오면 [onConnect](`/server/events?id=onconnect`)이벤트 콜백을 실행하고, 특정 연결이 서버로 데이터를 보낼 때 [onReceive](`/server/events?id=onreceive`) 함수를 호출합니다.

* 서버는 수천 개 이상의 클라이언트 연결을 동시에 가질 수 있으며, `$fd`는 클라이언트 연결의 고유 식별자입니다.
* `$server->send()` 메소드를 사용하여 클라이언트 연결에 데이터를 보냅니다. 인수는 `$fd` 클라이언트 식별자입니다.
* `$server->close()` 메소드를 사용하여 특정 클라이언트 연결을 강제로 닫을 수 있습니다.
* 클라이언트가 연결을 끊을 수도 있습니다. 이 경우 [onClose](`/server/events?id=onclose`) 이벤트 콜백이 트리거됩니다.

## 프로그램 실행

```shell
php tcpServer.php
```

`server.php` 프로그램을 명령 줄에서 실행한 다음 성공적으로 시작되면 `netstat` 도구를 사용하여 이미 `9501` 포트에서 수신되어 있는 것을 볼 수 있습니다.

이제 `telnet/netcat` 도구를 사용하여 서버에 연결할 수 있습니다.

```shell
telnet 127.0.0.1 9501
hello
Server: hello
```

## 서버 연결 문제 간단한 확인 방법

* `Linux`에서는 `netstat -an | grep 포트`를 사용하여 포트가 이미 'Listening' 상태인지 확인합니다.
* 이전 단계를 확인한 후에 방화벽 문제를 다시 확인하십시오.
* 서버에서 사용하는 IP 주소를 유의하십시오. 만약 127.0.0.1 루프백 주소를 사용하는 경우 클라이언트는 127.0.0.1을 사용하여 연결해야 합니다.
* 알리클라우드 서비스 또는 텐센트 클라우드를 사용하는 경우 보안 그룹에서 개발 포트를 설정해야 합니다.

## TCP 데이터 패킷 경계 문제

[TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터-패킷-경계-문제)를 참조하십시오.
