# Swoole\Server\Port

`Swoole\Server\Port`에 대한 자세한 설명입니다.

## 속성

### $host
리스닝하는 호스트 주소를 반환합니다. 이 속성은 `string` 유형의 문자열입니다.

```php
Swoole\Server\Port->host
```

### $port
리스닝하는 호스트 포트를 반환합니다. 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Port->port
```

### $type
이 그룹의 `server` 유형을 반환합니다. 이 속성은 열거형으로, `SWOOLE_TCP`, `SWOOLE_TCP6`, `SWOOLE_UDP`, `SWOOLE_UDP6`, `SWOOLE_UNIX_DGRAM`, `SWOOLE_UNIX_STREAM` 중 하나를 반환합니다.

```php
Swoole\Server\Port->type
```

### $sock
리스닝 소켓을 반환합니다. 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Port->sock
```

### $ssl
`ssl` 암호화를 사용하는지 여부를 반환합니다. 이 속성은 `bool` 유형입니다.

```php
Swoole\Server\Port->ssl
```

### $setting
이 포트에 대한 설정을 반환합니다. 이 속성은 배열 (`array`) 타입입니다.

```php
Swoole\Server\Port->setting
```

### $connections
이 포트에 연결된 모든 연결을 반환합니다. 이 속성은 이터레이터입니다.

```php
Swoole\Server\Port->connections
```

## 메서드

### set() 

`Swoole\Server\Port`의 런타임 매개변수를 설정하는 데 사용됩니다. 사용 방법은 [Swoole\Server->set()](/server/methods?id=set)와 동일합니다.

```php
Swoole\Server\Port->set(array $setting): void
```

### on() 

`Swoole\Server\Port` 콜백 함수를 설정하는 데 사용됩니다. 사용 방법은 [Swoole\Server->on()](/server/methods?id=on)와 동일합니다.

```php
Swoole\Server\Port->on(string $event, callable $callback): bool
```

### getCallback() 

설정된 콜백 함수를 반환합니다.

```php
Swoole\Server\Port->getCallback(string $name): ?callback
```

  * **매개변수**

    * `string $name`

      * 기능: 콜백 이벤트 이름
      * 기본값: 없음
      * 다른 값: 없음

  * **반환값**

    * 콜백 함수를 반환하면 작업이 성공적으로 수행되었음을 나타내고, `null`을 반환하면 해당 콜백 함수가 존재하지 않음을 나타냅니다.


### getSocket() 

현재 소켓 `fd`를 php의 `Socket` 객체로 변환합니다.

```php
Swoole\Server\Port->getSocket(): Socket|false
```

  * **반환값**

    * `Socket` 객체를 반환하면 작업이 성공적으로 수행되었음을 나타내고, `false`를 반환하면 작업이 실패했음을 나타냅니다.

!> 주의: `Swoole`을 컴파일할 때 `--enable-sockets`가 활성화되어 있어야만이 함수를 사용할 수 있습니다.
