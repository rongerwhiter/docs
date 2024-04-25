# Swoole\Server\Event

`Swoole\Server\Event`에 대한 자세한 설명입니다.

## 속성

### $reactor_id
`Reactor` 스레드 ID를 반환합니다. 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Event->reactor_id
```

### $fd
이 연결의 파일 설명자 `fd`를 반환합니다. 이 속성은 `int` 유형의 정수입니다.

```php
Swoole\Server\Event->fd
```

### $dispatch_time
요청 데이터 도착 시간인 `dispatch_time`을 반환합니다. 이 속성은 `double` 유형입니다. `onReceive` 이벤트에서만 속성이  `0`이 아닙니다.

```php
Swoole\Server\Event->dispatch_time
```

### $data
클라이언트가 보낸 데이터 `data`를 반환합니다. 이 속성은 `string` 유형의 문자열입니다. `onReceive` 이벤트에서만 속성이 `null`이 아닙니다.

```php
Swoole\Server\Event->data
```
