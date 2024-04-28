# 함수 별칭 요약

## 코루틴 단축 이름

코루틴 관련 `API`의 이름을 단순화합니다. `php.ini`를 수정하여 `swoole.use_shortname=On/Off`을 설정하여 단축명을 활성화/비활성화할 수 있습니다. 기본적으로 활성화되어 있습니다.

모든 `Swoole\Coroutine` 접두어 클래스 이름은 `Co`로 매핑됩니다. 또한 다음과 같은 매핑이 있습니다:

### 코루틴 만들기

```php
//Swoole\Coroutine::create은 go 함수와 동일합니다
go(function () {
	Co::sleep(0.5);
	echo 'hello';
});
go('test');
go([$object, 'method']);
```

### 채널 조작

```php
//Coroutine\Channel은 chan으로 줄여 쓸 수 있습니다
$c = new chan(1);
$c->push($data);
$c->pop();
```

### 지연 실행

```php
//Swoole\Coroutine::defer을 바로 defer로 사용할 수 있습니다
defer(function () use ($db) {
    $db->close();
});
```

## 단축 이름 방법

!> `go` 및 `defer`는 Swoole 버전 `v4.6.3` 이상에서 사용 가능합니다.

```php
use function Swoole\Coroutine\go;
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\defer;

run(function () {
    defer(function () {
        echo "co1 end\n";
    });
    sleep(1);
    go(function () {
        usleep(100000);
        defer(function () {
            echo "co2 end\n";
        });
        echo "co2\n";
    });
    echo "co1\n";
});
```

## 코루틴 시스템 API

`4.4.4` 버전에서 시스템 작업 관련 코루틴 `API`가 `Swoole\Coroutine` 클래스에서 `Swoole\Coroutine\System` 클래스로 이동되었습니다. 새 모듈로 독립되었습니다. 하위 호환성을 유지하기 위해 기존에는 코루틴 클래스 상단에 있는 별칭 메서드를 유지합니다.

* 예를 들어 `Swoole\Coroutine::sleep`은 `Swoole\Coroutine\System::sleep`와 대응됩니다.
* 예를 들어 `Swoole\Coroutine::fgets`는 `Swoole\Coroutine\System::fgets`와 대응됩니다.

## 클래스 단축 별칭 매핑

!> 네임스페이스 스타일 사용을 권장합니다.

| 밑줄 클래스 이름 스타일            | 네임스페이스 스타일                |
| --------------------------- | --------------------------- |
| swoole_server               | Swoole\Server               |
| swoole_client               | Swoole\Client               |
| swoole_process              | Swoole\Process              |
| swoole_timer                | Swoole\Timer                |
| swoole_table                | Swoole\Table                |
| swoole_lock                 | Swoole\Lock                 |
| swoole_atomic               | Swoole\Atomic               |
| swoole_atomic_long          | Swoole\Atomic\Long          |
| swoole_buffer               | Swoole\Buffer               |
| swoole_redis                | Swoole\Redis                |
| swoole_error                | Swoole\Error                |
| swoole_event                | Swoole\Event                |
| swoole_http_server          | Swoole\Http\Server          |
| swoole_http_client          | Swoole\Http\Client          |
| swoole_http_request         | Swoole\Http\Request         |
| swoole_http_response        | Swoole\Http\Response        |
| swoole_websocket_server     | Swoole\WebSocket\Server     |
| swoole_connection_iterator  | Swoole\Connection\Iterator  |
| swoole_exception            | Swoole\Exception            |
| swoole_http2_request        | Swoole\Http2\Request        |
| swoole_http2_response       | Swoole\Http2\Response       |
| swoole_process_pool         | Swoole\Process\Pool         |
| swoole_redis_server         | Swoole\Redis\Server         |
| swoole_runtime              | Swoole\Runtime              |
| swoole_server_port          | Swoole\Server\Port          |
| swoole_server_task          | Swoole\Server\Task          |
| swoole_table_row            | Swoole\Table\Row            |
| swoole_timer_iterator       | Swoole\Timer\Iterator       |
| swoole_websocket_closeframe | Swoole\Websocket\Closeframe |
| swoole_websocket_frame      | Swoole\Websocket\Frame      |
