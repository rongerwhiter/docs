# Swoole\Server\Packet

`Swoole\Server\Packet`에 대한 자세한 설명입니다.

## 속성

### $server_socket
서버 소켓 파일 디스크립터 `fd`를 반환하며, 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Packet->server_socket
```

### $server_port
서버 리스닝 포트 `server_port`를 반환하며, 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Packet->server_port
```

### $dispatch_time
요청 데이터 도착 시간인 `dispatch_time`을 반환하며, 이 속성은 `double` 유형입니다.

```php
Swoole\Server\Packet->dispatch_time
```

### $address
클라이언트 주소인 `address`를 반환하며, 이 속성은 `string` 유형의 문자열입니다.

```php
Swoole\Server\Packet->address
```

### $port
클라이언트 리스닝 포트인 `port`를 반환하며, 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Packet->port
```

### $data
클라이언트가 전달한 데이터인 `data`를 반환하며, 이 속성은 `string` 유형의 문자열입니다.

```php
Swoole\Server\Packet->data
```
