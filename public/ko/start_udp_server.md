# UDP 서버

## 프로그램 코드

다음 코드를 udpServer.php에 작성하십시오.

```php
$server = new Swoole\Server('127.0.0.1', 9502, SWOOLE_PROCESS, SWOOLE_SOCK_UDP);

// 데이터 수신 이벤트를 청취합니다.
$server->on('Packet', function ($server, $data, $clientInfo) {
    var_dump($clientInfo);
    $server->sendto($clientInfo['address'], $clientInfo['port'], "Server：{$data}");
});

// 서버 시작
$server->start();
```

UDP 서버는 TCP 서버와 다르며, UDP는 연결 개념이 없습니다. 서버를 시작한 후, 클라이언트는 Connect할 필요 없이 바로 서버가 청취하는 9502 포트에 데이터 패킷을 전송할 수 있습니다. 관련 이벤트는 onPacket입니다.

* `$clientInfo`는 클라이언트의 관련 정보이며, 클라이언트의 IP 및 포트 등을 포함한 배열입니다.
* `$server->sendto` 메서드를 사용하여 클라이언트에 데이터를 전송합니다.
!> Docker는 기본적으로 TCP 프로토콜을 사용하여 통신하므로, UDP 프로토콜을 사용해야 하는 경우 Docker 네트워크를 설정해야 합니다.  
```shell
docker run -p 9502:9502/udp <image-name>
```

## 서비스 시작

```shell
php udpServer.php
```

UDP 서버는 `netcat -u`를 사용하여 연결을 테스트할 수 있습니다.

```shell
netcat -u 127.0.0.1 9502
hello
Server: hello
```
