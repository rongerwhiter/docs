# 이벤트

`Swoole` 확장은 직접 하위 `epoll/kqueue` 이벤트 루프를 조작하는 인터페이스를 제공합니다. 다른 확장이 생성한 `socket`, `PHP` 코드에서 생성된 `stream/socket` 확장의 `socket`을 `Swoole`의 [EventLoop](/learn?id=이벤트루프란 무엇인가)에 추가할 수 있습니다.
그렇지 않으면 제3자의 $fd가 동기 IO인 경우, Swoole의 EventLoop가 실행되지 않을 수 있습니다. [참고 사례](/learn?id=동기io를 비동기io로 변환하는 예).

!> `Event` 모듈은 상당히 낮은 수준의 모듈이며, `epoll`의 간단한 래핑입니다. 사용자는 IO 다중화 프로그래밍 경험이 있어야 합니다.
## 이벤트 우선순위

1. `Process::signal`을 통해 설정된 신호 처리 콜백 함수
2. `Timer::tick` 및 `Timer::after`를 통해 설정된 타이머 콜백 함수
3. `Event::defer`를 통해 설정된 지연 실행 함수
4. `Event::cycle`를 통해 설정된 주기적 콜백 함수
그것은 코드 블록 안에 있으므로 번역하지 않습니다.
### add()

`socket`를 기본 `reactor` 이벤트 감지기에 추가합니다. 이 함수는 `Server` 또는 `Client` 모드에서 사용할 수 있습니다.
```php
Swoole\Event::add(mixed $sock, callable $read_callback, callable $write_callback = null, int $flags = null): bool
```

!> `Server` 프로그램에서는 `Worker` 프로세스가 시작된 후에 사용해야 합니다. `Server::start` 이전에는 어떤 비동기 `IO` 인터페이스도 호출해서는 안됩니다.

* **매개변수** 

  * **`mixed $sock`**
    * **기능**: 파일 기술자, `stream` 리소스, `sockets` 리소스, object
    * **기본값**: 없음
    * **다른 값**: 없음

  * **`callable $read_callback`**
    * **기능**: 읽기 가능 이벤트 콜백 함수
    * **기본값**: 없음
    * **다른 값**: 없음

  * **`callable $write_callback`**
    * **기능**: 쓰기 가능 이벤트 콜백 함수【이 매개변수는 문자열 함수 이름, object + 메소드, 클래스의 정적 메소드 또는 무명 함수일 수 있습니다. 이 `socket`이 읽기 또는 쓰기 가능할 때 해당 함수를 호출합니다.】
    * **기본값**: 없음
    * **다른 값**: 없음

  * **`int $flags`**
    * **기능**: 이벤트 유형의 마스크【종료/시작 가능한 읽기 또는 쓰기 이벤트를 선택할 수 있습니다. 예: `SWOOLE_EVENT_READ`, `SWOOLE_EVENT_WRITE` 또는 `SWOOLE_EVENT_READ|SWOOLE_EVENT_WRITE`】
    * **기본값**: 없음
    * **다른 값**: 없음

* **$sock 4가지 타입**

타입 | 설명
---|---
int | 파일 기술자, `Swoole\Client->$sock`, `Swoole\Process->$pipe` 및 기타 fd 포함
stream 리소스 | `stream_socket_client`/`fsockopen`으로 생성된 리소스
sockets 리소스 | `sockets` 확장을 통해 생성된 `socket_create` 리소스. 컴파일 시 `./configure --enable-sockets`를 지정해야 함
object | `Swoole\Process` 또는 `Swoole\Client`, 하위 레벨에서 [UnixSocket](/learn?id=IPC란 무엇인가)(`Process`) 또는 클라이언트 연결의 `socket`(`Swoole\Client`)으로 자동 변환

* **반환 값**

  * 이벤트 감지 성공적으로 추가 시 `true` 반환
  * 실패할 경우 `false` 반환, 오류 코드는 `swoole_last_error`를 사용해야 함
  * 이미 추가된 `socket`은 중복으로 추가할 수 없으며 `swoole_event_set`을 사용하여 `socket`에 대한 콜백 함수 및 이벤트 유형을 수정할 수 있음

  !> `Swoole\Event::add`를 사용하여 `socket`를 이벤트 감지에 추가하면 하위 수준에서 해당 `socket`이 블로킹 모드로 설정됩니다.

* **사용 예시**

```php
$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

Swoole\Event::add($fp, function($fp) {
    $resp = fread($fp, 8192);
    //소켓 처리 완료 후 이벤트 감지기에서 소켓 제거
    Swoole\Event::del($fp);
    fclose($fp);
});
echo "Finish\n";  //Swoole\Event::add는 프로세스를 차단하지 않으며, 이 코드는 순차적으로 실행됩니다
```

* **콜백 함수**

  * 읽기 가능 `($read_callback)` 이벤트 콜백 함수에서는 `fread`, `recv` 등의 함수를 사용하여 소켓 버퍼 영역에서 데이터를 읽어야 합니다. 그렇지 않으면 이벤트가 계속해서 트리거됩니다. 계속해서 읽지 않으려면 `Swoole\Event::del`을 사용하여 이벤트 감시를 제거해야 합니다.
  * 쓰기 가능 `($write_callback)` 이벤트 콜백 함수에서 소켓에 작성한 후에는 반드시 `Swoole\Event::del`를 호출하여 이벤트 감시를 제거해야 합니다. 그렇지 않으면 쓰기 가능 이벤트가 계속해서 트리거됩니다.
  * `fread`, `socket_recv`, `socket_read`, `Swoole\Client::recv`를 실행하고 반환값이 `false`이며 오류 코드가 `EAGAIN`인 경우 현재 소켓 수신 버퍼에 데이터가 없음을 나타냅니다. 이 경우 읽기 가능한 이벤트를 대기하도록 해야 합니다[EventLoop](/learn?id=이벤트루프가 무엇인가) 통지
  * `fwrite`, `socket_write`, `socket_send`, `Swoole\Client::send` 작업을 실행하고 반환값이 `false`이며 오류 코드가 `EAGAIN`인 경우 현재 소켓 전송 버퍼가 가득차 있어 일시적으로 데이터를 전송할 수 없음을 나타냅니다. 쓰기 가능한 이벤트를 수신하도록 대기해야 합니다[EventLoop](/learn?id=이벤트루프가 무엇인가) 통지
### set()

이벤트 리스너의 콜백 함수와 마스크를 변경합니다.

```php
Swoole\Event::set($fd, mixed $read_callback, mixed $write_callback, int $flags): bool
```

* **매개변수** 

  * [Event::add](/event?id=add)와 동일한 매개변수입니다. 만약 전달된 `$fd`가 [EventLoop](/learn?id=什게 eventloop)에 존재하지 않는 경우 `false`를 반환합니다.
  * `$read_callback`이 `null`이 아닌 경우, 지정된 함수로 읽을 수 있는 이벤트 콜백 함수를 변경합니다.
  * `$write_callback`이 `null`이 아닌 경우, 지정된 함수로 쓸 수 있는 이벤트 콜백 함수를 변경합니다.
  * `$flags`로는 읽을 수 있는 (`SWOOLE_EVENT_READ`) 및 쓸 수 있는 (`SWOOLE_EVENT_WRITE`) 이벤트를 감시하는 것을 활성화/비활성화할 수 있습니다.

  !> 주의: `SWOOLE_EVENT_READ` 이벤트를 감시하지만 현재 `read_callback`이 설정되어 있지 않은 경우, 하위 수준에서는 `false`를 직접 반환하고 추가 실패합니다. `SWOOLE_EVENT_WRITE`도 동일합니다.

* **상태 변경**

  * 읽을 수 있는 이벤트 콜백을 설정하였지만 `SWOOLE_EVENT_READ` 읽을 수 있는 이벤트를 감시하지 않았을 때, 하위 수준은 콜백 함수 정보만 저장하고 어떠한 이벤트 콜백도 발생시키지 않습니다.
  * `Event::set($fd, null, null, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)`를 사용하여 수정된 이벤트 유형을 트리거할 수 있습니다. 이 경우, 하위 수준은 읽을 수 있는 이벤트를 발생시킵니다.

* **콜백 함수 해제**

!> `Event::set`은 콜백 함수를 대체할 수 있지만 이벤트 콜백 함수를 해제할 수 없습니다. 즉, `Event::set($fd, null, null, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)`를 사용하여 콜백 함수를 `null`로 설정할 수는 없습니다. 이는 `Event::add`로 설정된 콜백 함수를 변경하지 않고 이벤트 콜백 함수를 `null`로 설정하는 것을 의미합니다.

이벤트 감시를 지우는 `Event::del`을 호출할 때에만 하위 수준에서는 `read_callback` 및 `write_callback` 이벤트 콜백 함수를 해제합니다.
### isset()

전달된 `$fd`가 이미 이벤트 리스너에 추가되었는지 확인합니다.

```php
Swoole\Event::isset(mixed $fd, int $events = SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE): bool
```

* **파라미터** 

  * **`mixed $fd`**
    * **기능**: 임의의 소켓 파일 디스크립터【참조 [Event::add](/event?id=add) 문서】
    * **기본값**: 없음
    * **다른 값**: 없음

  * **`int $events`**
    * **기능**: 확인할 이벤트 유형
    * **기본값**: 없음
    * **다른 값**: 없음

* **$events**

이벤트 유형 | 설명
---|---
`SWOOLE_EVENT_READ` | 읽기 이벤트를 청취 중인지 확인
`SWOOLE_EVENT_WRITE` | 쓰기 이벤트를 청취 중인지 확인
`SWOOLE_EVENT_READ \| SWOOLE_EVENT_WRITE` | 읽기 또는 쓰기 이벤트를 청취 중인지 확인

* **사용 예시**

```php
use Swoole\Event;

$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

Event::add($fp, function($fp) {
    $resp = fread($fp, 8192);
    Swoole\Event::del($fp);
    fclose($fp);
}, null, SWOOLE_EVENT_READ);
var_dump(Event::isset($fp, SWOOLE_EVENT_READ)); // true 반환
var_dump(Event::isset($fp, SWOOLE_EVENT_WRITE)); // false 반환
var_dump(Event::isset($fp, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)); // true 반환
```
### write()

PHP의 내장 `stream/sockets` 확장을 사용하여 생성된 소켓에 데이터를 보내기 위해 `fwrite/socket_send` 등의 함수를 사용합니다. 데이터 양이 많을 때 소캣 쓰기 버퍼가 가득 차면 데이터를 보낼 때 블로킹되거나 [EAGAIN](/other/errno?id=linux) 오류가 발생할 수 있습니다.

`Event::write` 함수는 `stream/sockets` 리소스의 데이터 전송을 **비동기**로 만들어줍니다. 버퍼가 가득 차거나 [EAGAIN](/other/errno?id=linux)이 반환되면 Swoole은 데이터를 보내는 큐에 추가하고 쓰기 가능한 것을 확인하고 자동으로 데이터를 쓸 수 있도록 합니다.

```php
Swoole\Event::write(mixed $fd, mixed $data): bool
```

* **파라미터** 

  * **`mixed $fd`**
    * **기능**：임의의 소켓 파일 디스크립터【참조 [Event::add](/event?id=add) 문서】
    * **기본값**：없음
    * **기타**：없음

  * **`mixed $data`**
    * **기능**：전송할 데이터 【전송 데이터 길이는 `Socket` 버퍼 크기를 초과해서는 안 됨】
    * **기본값**：없음
    * **기타**：없음

!> `Event::write`는 `SSL/TLS`와 같이 터널 암호화된 `stream/sockets` 리소스에는 사용할 수 없습니다.  
`Event::write` 작업이 성공하면 해당 `$socket`이 블로킹 모드가 아닌 것으로 자동 설정됩니다.

* **사용 예시**

```php
use Swoole\Event;

$fp = stream_socket_client('tcp://127.0.0.1:9501');
$data = str_repeat('A', 1024 * 1024*2);

Event::add($fp, function($fp) {
     echo fread($fp);
});

Event::write($fp, $data);
```
#### SOCKET 버퍼가 꽉 찬 경우, Swoole의 하부 논리

지속적으로 `SOCKET`에 쓰기를 수행하다 상대편이 읽기를 충분히 빠르게 처리하지 못하면, `SOCKET` 버퍼가 가득 찰 수 있습니다. `Swoole`의 하부 논리는 데이터를 메모리 버퍼에 보관하고, 쓰기 가능한 이벤트가 발생할 때까지 `SOCKET`로 기록하지 않습니다.

만약 메모리 버퍼도 가득 차게 되면, 이때 `Swoole`의 하부는 `pipe buffer overflow, reactor will block.` 오류를 발생시키고 블로킹 대기 상태로 들어갑니다.

!> 버퍼가 가득 찼을 때 `false`를 반환하는 것은 원자적인 동작이며, 모두 성공적으로 쓰기를 완료하거나 전부 실패하는 경우만 발생할 것입니다.
### del()

`reactor`에서 `socket`의 리스너를 제거합니다. `Event::del`은 `Event::add`와 짝을 이루어 사용해야 합니다.

```php
Swoole\Event::del(mixed $sock): bool
```

!> `socket`의 `close` 작업 전에 `Event::del`을 사용하여 이벤트 리스너를 제거해야 합니다. 그렇지 않으면 메모리 누수가 발생할 수 있습니다.

* **매개변수**

  * **`mixed $sock`**
    * **기능**: `socket`의 파일 설명자
    * **기본 값**: 없음
    * **기타 값**: 없음
### exit()

이벤트 루프를 종료합니다.

!> 이 함수는 `Client` 프로그램에서만 유효합니다

```php
Swoole\Event::exit(): void
```
### defer()

다음 이벤트 루프가 시작될 때 함수를 실행합니다.

```php
Swoole\Event::defer(mixed $callback_function);
```

!> `Event::defer`의 콜백 함수는 현재 `EventLoop`의 이벤트 루프가 끝나고 다음 이벤트 루프가 시작하기 전에 실행됩니다.

* **매개변수**

  * **`mixed $callback_function`**
    * **기능**: 시간이 지난 후 실행될 함수입니다. (호출 가능해야 합니다. 콜백 함수는 어떤 매개변수도 받지 않으며, 익명 함수의 `use` 구문을 사용하여 콜백 함수로 매개변수를 전달할 수 있습니다. `$callback_function` 함수가 실행되는 동안 새로운 `defer` 작업을 추가하더라도 이번 이벤트 루프 내에서 실행이 완료됩니다.)
    * **기본값**: 없음
    * **다른 값**: 없음

* **사용 예시**

```php
Swoole\Event::defer(function(){
    echo "After EventLoop\n";
});
```
### cycle()

이벤트 루프 주기적으로 실행할 함수를 정의합니다. 이 함수는 각 이벤트 루프가 끝날 때 호출됩니다.

```php
Swoole\Event::cycle(callable $callback, bool $before = false): bool
```

* **매개변수** 

  * **`callable $callback_function`**
    * **기능**: 설정할 콜백 함수 【`$callback`이 `null`이면 `cycle` 함수를 제거합니다. 이미 설정된 `cycle` 함수를 다시 설정하면 이전 설정이 덮어씌워집니다.】
    * **기본값**: 없음
    * **기타 값**: 없음

  * **`bool $before`**
    * **기능**: [EventLoop](/learn?id=什게-is-eventloop) 이전에 이 함수를 호출합니다.
    * **기본값**: 없음
    * **기타 값**: 없음

!> `before=true`와 `before=false` 두 콜백 함수가 동시에 존재할 수 있습니다.

  * **사용 예시**

```php
Swoole\Timer::tick(2000, function ($id) {
    var_dump($id);
});

Swoole\Event::cycle(function () {
    echo "안녕하세요 [1]\n";
    Swoole\Event::cycle(function () {
        echo "안녕하세요 [2]\n";
        Swoole\Event::cycle(null);
    });
});
```
### wait()

이벤트 리스너를 시작합니다.

!> 이 함수를 PHP 프로그램 맨 아래에 놓으십시오.

```php
Swoole\Event::wait();
```

* **사용 예시**

```php
Swoole\Timer::tick(1000, function () {
    echo "hello\n";
});

Swoole\Event::wait();
```
### dispatch()

이벤트 리스닝을 시작합니다.

!> `reactor->wait` 작업을 한 번만 실행하며, `Linux` 플랫폼에서는 `epoll_wait`를 수동으로 호출하는 것과 비슷합니다. `Event::dispatch`와 다른 점은 `Event::wait`가 내부적으로 루프를 유지한다는 것입니다.

```php
Swoole\Event::dispatch();
```

* **사용 예시**

```php
while(true)
{
    Event::dispatch();
}
```

이 함수의 목적은 `amp`와 같은 몇 가지 프레임워크와 호환성을 유지하기 위함입니다. 이 프레임워크에서는 내부적으로 `reactor`의 루프를 제어하지만, `Event::wait` 사용시 Swoole 하부에서 제어가 유지되므로 프레임워크에 양보할 수 없습니다.
