# 코루틴 TCP/UDP 클라이언트

`Coroutine\Client`는 `TCP`, `UDP`, [unixSocket](/learn?id=IPC이란) 전송 프로토콜에 대한 [Socket 클라이언트](/coroutine_client/socket) 래핑 코드를 제공하며, 사용 시에는 단순히 `new Swoole\Coroutine\Client`만 필요합니다.

* **구현 원리**

    * `Coroutine\Client`의 모든 네트워크 요청 관련 메서드는 `Swoole`이 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 수행하며, 비즈니스 계층에서는 인식할 필요가 없습니다.
    * 사용 방법 및 [Client](/client) 동기 모드 메서드와 완전히 호환됩니다.
    * `connect` 타임아웃은 `Connect`, `Recv`, `Send` 타임아웉에 동시 적용됩니다.

* **상속 관계**

    * `Coroutine\Client`와 [Client](/client)는 상속 관계가 아니지만, `Client`가 제공하는 모든 메서드는 `Coroutine\Client`에서도 사용할 수 있습니다. 자세한 내용은 [Swoole\Client](/client?id=메서드)를 참조하고, 여기에서는 다시 설명하지 않겠습니다.
    * `Coroutine\Client`에서는 `set` 메서드를 사용하여 [구성 옵션](/client?id=구성)을 설정할 수 있으며, 사용 방법은 `Client->set`과 완전히 동일합니다. 함수가 다른 점에 대해선 `set()` 함수 부분에서 개별적으로 설명됩니다.

* **사용 예시**

```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    if (!$client->connect('127.0.0.1', 9501, 0.5))
    {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    echo $client->recv();
    $client->close();
});
```

* **프로토콜 처리**

코루틴 클라이언트는 길이와 `EOF` 프로토콜 처리를 지원하며, 설정 방법은 [Swoole\Client](/client?id=구성)과 완전히 동일합니다.

```php
$client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
$client->set(array(
    'open_length_check'     => true,
    'package_length_type'   => 'N',
    'package_length_offset' => 0, // 패키지 길이의 값이 있는 N 번째 바이트
    'package_body_offset'   => 4, // 길이 계산이 시작되는 N 번째 바이트
    'package_max_length'    => 2000000, // 프로토콜 최대 길이
));
```
### connect()

원격 서버에 연결합니다.

```php
Swoole\Coroutine\Client->connect(string $host, int $port, float $timeout = 0.5): bool
```

  * **Arguments** 

    * **`string $host`**
      * **기능**：원격 서버 주소【하위 수준에서는 자동으로 코루틴 전환하여 도메인을 IP 주소로 해석합니다】
      * **기본값**：없음
      * **기타 값**：없음

    * **`int $port`**
      * **기능**：원격 서버 포트
      * **기본값**：없음
      * **기타 값**：없음

    * **`float $timeout`**
      * **기능**：네트워크 IO의 타임아웃 시간; `connect/send/recv`를 포함하며, 시간이 초과되면 연결이 자동으로 `close`됩니다. 자세한 내용은 [클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃-규칙)을 참조하세요.
      * **시간 단위**：초 【부동 소수점 지원, 예: `1.5`는 `1초` + `500ms`를 나타냅니다】
      * **기본값**：`0.5초`
      * **다른 값**：없음

* **힌트**

    * 연결 실패 시 `false`를 반환합니다.
    * 타임아웃이 발생하면 `cli->errCode`를 확인하여 `110`인지 확인하세요.

* **재시도 실패**

!> `connect`에 실패한 후에는 직접 재연결할 수 없습니다. 기존 `소켓`을 `close`로 닫고, 다시 `connect`를 시도해야 합니다.

```php
//연결 실패
if ($cli->connect('127.0.0.1', 9501) == false) {
    //기존 소켓을 닫고
    $cli->close();
    //재시도
    $cli->connect('127.0.0.1', 9501);
}
```

* **예시**

```php
if ($cli->connect('127.0.0.1', 9501)) {
    $cli->send('데이터');
} else {
    echo '연결 실패.';
}

if ($cli->connect('/tmp/rpc.sock')) {
    $cli->send('데이터');
} else {
    echo '연결 실패.';
}
```
### isConnected()

Client의 연결 상태를 반환합니다.

```php
Swoole\Coroutine\Client->isConnected(): bool
```

  * **Return Value**

    * `false`를 반환하면 현재 서버에 연결되지 않은 상태입니다.
    * `true`를 반환하면 현재 서버에 연결된 상태입니다.

!> `isConnected` 메소드의 반환 값은 응용 계층 상태를 나타냅니다. 이는 `Client`가 `Server`에 성공적으로 연결되었고 `close`를 통해 연결을 닫지 않은 것을 나타냅니다. `Client`는 `send` , `recv`, `close`와 같은 작업을 수행할 수 있지만 `connect`를 다시 실행할 수는 없습니다.  
이는 연결이 항상 사용 가능하다는 것을 의미하지는 않습니다. `send` 또는 `recv`를 실행할 때 오류가 발생할 수 있습니다. 응용 프로그램 계층은 하위 `TCP` 연결의 상태를 알 수 없으며, `send` 또는 `recv`를 실행할 때 응용 계층과 커널 간의 상호 작용이 발생해야 실제 연결 사용 가능 상태를 얻을 수 있습니다.
### send()

데이터를 보냅니다.

```php
Swoole\Coroutine\Client->send(string $data): int|bool
```

  * **매개변수** 

    * **`string $data`**
    
      * **기능**：보낼 데이터로, 문자열 형식이어야 하며, 바이너리 데이터를 지원합니다.
      * **기본값**：없음
      * **기타 값**：없음

  * 데이터를 성공적으로 보내면 쓰여진 `Socket` 버퍼 영역의 바이트 수가 반환되며, 하위 레벨에서는 가능한 한 모든 데이터를 보냅니다. 반환된 바이트 수가 전달된 `$data`의 길이와 다른 경우에는 `Socket`이 종닫힌 상태일 수 있으며, 다음 `send` 또는 `recv` 호출시 해당하는 오류 코드가 반환됩니다.

  * 전송에 실패하면 false가 반환되며, `$client->errCode`를 사용하여 오류 이유를 얻을 수 있습니다.
### recv()

recv 메서드는 서버로부터 데이터를 수신하는 데 사용됩니다.

```php
Swoole\Coroutine\Client->recv(float $timeout = 0): string|bool
```

  * **매개변수** 

    * **`float $timeout`**
      * **기능**：타임아웃 설정
      * **단위**：초【부동 소수점 지원, 예: `1.5`는 `1초`+`500밀리초`를 나타냄】
      * **기본값**：[클라이언트 타임아웃 규칙](/coroutine_client/init?id=타임아웃규칙)을 참고하세요
      * **다른 값**：없음

    !> 타임아웃 설정은 지정된 매개변수를 우선적으로 사용하고, 그 다음으로 `set` 메서드에서 전달된 `timeout` 구성을 사용합니다. 발생한 타임아웃 오류 코드는 `ETIMEDOUT`입니다.

  * **반환 값**

    * [통신 프로토콜](/client?id=프로토콜해석)을 설정한 경우, `recv`는 완전한 데이터를 반환하며, 길이는 [package_max_length](/server/setting?id=package_max_length)에 제한됩니다.
    * 통신 프로토콜을 설정하지 않은 경우, `recv`는 최대 `64K` 데이터를 반환합니다.
    * 통신 프로토콜을 설정하지 않은 경우, 원시 데이터를 반환하며, PHP 코드에서 네트워크 프로토콜 처리를 직접 구현해야 합니다.
    * `recv`가 빈 문자열을 반환하면 서버가 연결을 닫았음을 나타내며, `close`를 해야 합니다.
    * `recv`가 실패하면 `false`를 반환하며, `$client->errCode`를 확인하여 오류 원인을 파악하고, 아래 [완전한 예제](/coroutine_client/client?id=완전한예제)를 참고하여 처리할 수 있습니다.
### close()

연결을 닫습니다.

!> `close` 함수는 차단되지 않고 즉시 반환됩니다. 연결을 닫는 작업 중에는 코루틴 전환이 발생하지 않습니다.

```php
Swoole\Coroutine\Client->close(): bool
```
### peek()

데이터를 엿보다.

!> `peek` 메소드는 `socket`을 직접 조작하기 때문에 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 일으키지 않습니다.

```php
Swoole\Coroutine\Client->peek(int $length = 65535): string
```

  * **팁**

    * `peek` 메소드는 내부 커널 `socket` 버퍼 영역을 엿보는 데만 사용되며 오프셋을 변경하지 않습니다. `peek`를 사용한 후에 `recv`를 호출하여 이 부분 데이터를 여전히 읽을 수 있습니다.
    * `peek` 메소드는 논블로킹이며 즉시 반환됩니다. `socket` 버퍼 영역에 데이터가 있는 경우 해당 데이터 내용이 반환됩니다. 버퍼가 비어 있으면 `false`가 반환되고 `$client->errCode`가 설정됩니다.
    * 연결이 닫혀 있으면 `peek`는 빈 문자열을 반환합니다.
### set()

클라이언트 매개변수를 설정합니다.

```php
Swoole\Coroutine\Client->set(array $settings): bool
```

* **설정 매개변수**

    * [Swoole\Client](/client?id=set)를 참조하십시오.

* **[Swoole\Client](/client?id=set)와의 차이점**

    코루틴 클라이언트는 더 세분화된 시간 초과 제어를 제공합니다. 다음을 설정할 수 있습니다:
    
    * `timeout` : 연결, 송신, 수신 시간 모두를 포함한 총 시간 초과
    * `connect_timeout` : 연결 시간 초과
    * `read_timeout` : 수신 시간 초과
    * `write_timeout` : 송신 시간 초과
    * [클라이언트 시간 초과 규칙](/coroutine_client/init?id=超时规则)을 참조하십시오.

* **예제**

```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    $client->set(array(
        'timeout' => 0.5,
        'connect_timeout' => 1.0,
        'write_timeout' => 10.0,
        'read_timeout' => 0.5,
    ));

    if (!$client->connect('127.0.0.1', 9501, 0.5))
    {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    echo $client->recv();
    $client->close();
});
```
```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    if (!$client->connect('127.0.0.1', 9501, 0.5)) {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    while (true) {
        $data = $client->recv();
        if (strlen($data) > 0) {
            echo $data;
            $client->send(time() . PHP_EOL);
        } else {
            if ($data === '') {
                // 모두 빈 문자열인 경우 연결을 닫습니다.
                $client->close();
                break;
            } else {
                if ($data === false) {
                    // 업무 로직 및 오류 코드에 따라 직접 처리할 수 있습니다. 예를 들어:
                    // 시간 초과일 때는 연결을 닫지 않고 다른 경우에는 연결을 닫습니다.
                    if ($client->errCode !== SOCKET_ETIMEDOUT) {
                        $client->close();
                        break;
                    }
                } else {
                    $client->close();
                    break;
                }
            }
        }
        \Co::sleep(1);
    }
});
```
