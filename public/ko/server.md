# TCP 서버

?> `Swoole\Coroutine\Server`는 완전히 [코루틴](/coroutine)화된 클래스로, 코루틴 `TCP` 서버를 생성하는데 사용됩니다. TCP 및 [unixSocket](/learn?id=IPC) 유형을 지원합니다.

[Server](/server/tcp_init) 모듈과의 차이점:

* 동적 생성 및 소멸, 런타임에서 포트를 동적으로 수신 대기하고 서버를 동적으로 닫을 수 있음
* 연결 처리 과정은 완전히 동기적이며 프로그램은 `Connect`、`Receive`、`Close` 이벤트를 순차적으로 처리할 수 있음

!> 4.4 버전 이상에서 사용 가능
## 짧은 이름

`Co\Server`를 사용할 수 있습니다.
Please provide me with the text you would like to have translated or analyzed.
### __construct()

?> **생성자 메서드.** 

```php
Swoole\Coroutine\Server::__construct(string $host, int $port = 0, bool $ssl = false, bool $reuse_port = false);
```

  * **매개변수** 

    * **`string $host`**
      * **기능**：듣기 주소
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $port`**
      * **기능**：듣기 포트【0이면 운영 체제가 무작위 포트를 할당합니다】
      * **기본값**：없음
      * **다른 값**：없음

    * **`bool $ssl`**
      * **기능**：SSL 암호화를 활성화할지 여부
      * **기본값**：`false`
      * **다른 값**：`true`

    * **`bool $reuse_port`**
      * **기능**：포트 재사용을 활성화할지 여부, 이는 [이 섹션](/server/setting?id=enable_reuse_port)에서의 설정과 동일한 효과를 냅니다
      * **기본값**：`false`
      * **다른 값**：`true`
      * **버전 영향**：Swoole 버전 >= v4.4.4

  * **팁**

    * **$host 매개변수는 3가지 형식을 지원합니다**

      * `0.0.0.0/127.0.0.1`: IPv4 주소
      * `::/::1`: IPv6 주소
      * `unix:/tmp/test.sock`: [UnixSocket](/learn?id=IPC가-뭔가요) 주소

    * **예외**

      * 잘못된 매개변수, 주소 및 포트 바인딩 실패, 'listen' 실패 시 `Swoole\Exception` 예외가 발생합니다.
### set()

?> **프로토콜 처리 매개변수를 설정합니다.**

```php
Swoole\Coroutine\Server->set(array $options);
```

  * **구성 매개변수**

    * 매개변수`$options`는 반드시 일차원 연관 배열이어야 하며, [setprotocol](/coroutine_client/socket?id=setprotocol) 메서드가 수락하는 구성 항목과 완전히 동일해야 합니다.

    !> 반드시 [start()](/coroutine/server?id=start) 메서드 이전에 매개변수를 설정해야 합니다.

    * **길이 프로토콜**

    ```php
    $server = new Swoole\Coroutine\Server('127.0.0.1', $port, $ssl);
    $server->set([
      'open_length_check' => true,
      'package_max_length' => 1024 * 1024,
      'package_length_type' => 'N',
      'package_length_offset' => 0,
      'package_body_offset' => 4,
    ]);
    ```

    * **SSL 인증서 설정**

    ```php
    $server->set([
      'ssl_cert_file' => dirname(__DIR__) . '/ssl/server.crt',
      'ssl_key_file' => dirname(__DIR__) . '/ssl/server.key',
    ]);
    ```
### handle()

?> **연결 처리 함수를 설정합니다.**

!> 반드시 [start()](/coroutine/server?id=start) 이전에 처리 함수를 설정해야 합니다.

```php
Swoole\Coroutine\Server->handle(callable $fn);
```

  * **매개변수** 

    * **`callable $fn`**
      * **기능** : 연결 처리 함수를 설정합니다.
      * **기본값** : 없음
      * **기타 값** : 없음
      
  * **예시** 

    ```php
    $server->handle(function (Swoole\Coroutine\Server\Connection $conn) {
        while (true) {
            $data = $conn->recv();
        }
    });
    ```

    !> - 서버가 `Accept` (연결 수립) 성공 후, 자동으로 [코루틴](/coroutine?id=코루틴 스케줄러)을 생성하고 `$fn`을 실행합니다.
    - `$fn`은 새로운 하위 코루틴 공간에서 실행되므로 함수 내에서 다시 코루틴을 생성할 필요가 없습니다.
    - `$fn`은 [Swoole\Coroutine\Server\Connection](/coroutine/server?id=coroutineserverconnection) 객체 유형의 매개변수를 수신합니다.
    - 현재 연결의 소켓 객체를 얻으려면 [exportSocket()](/coroutine/server?id=exportsocket)을 사용할 수 있습니다.
### shutdown()

?> **서버를 종료합니다.**

?> 밑바닥에서 `start`와 `shutdown`을 여러 번 호출할 수 있습니다.

```php
Swoole\Coroutine\Server->shutdown(): bool
```
### start()

?> **서버를 시작합니다.**

```php
Swoole\Coroutine\Server->start(): bool
```

  * **반환 값**

    * 실패할 경우 `false`를 반환하고 `errCode` 속성을 설정합니다.
    * 성공하면 루프로 진입하고 `Accept` 연결합니다.
    * `Accept` (연결 만들기) 후에는 새로운 코루틴이 생성되며 해당 코루틴에서는 지정된 함수를 호출하도록 합니다.

  * **에러 처리**

    * `Accept`(연결 만들기) 중에 `Too many open file` 오류가 발생하거나 서브 코루틴을 만들 수 없는 경우, `1`초 일시 중지한 후 `Accept`을 계속합니다.
    * 오류가 발생하면 `start()` 메소드가 반환되며, 오류 정보는 `Warning` 형식으로 표시됩니다.
## 객체
### Coroutine\Server\Connection

`Swoole\Coroutine\Server\Connection` 객체는 네 가지 메서드를 제공합니다.
#### recv()

데이터를 받습니다. 프로토콜 처리가 설정되어 있다면 매번 완전한 패킷을 반환합니다.

```php
function recv(float $timeout = 0)
```  
#### send()

데이터를 전송합니다.

```php
function send(string $data)
```
```php
function close(): bool
```

연결을 닫습니다.
```
#### exportSocket()

현재 연결된 Socket 객체를 반환합니다. 더 많은 하부 메서드를 호출할 수 있습니다. 자세한 내용은 [Swoole\Coroutine\Socket](/coroutine_client/socket)를 참조하십시오.

```php
function exportSocket(): Swoole\Coroutine\Socket
```
```php
use Swoole\Process;
use Swoole\Coroutine;
use Swoole\Coroutine\Server\Connection;

//다중 프로세스 관리 모듈
$pool = new Process\Pool(2);
//각 OnWorkerStart 콜백에서 코루틴을 자동으로 생성합니다.
$pool->set(['enable_coroutine' => true]);
$pool->on('workerStart', function ($pool, $id) {
    //각 프로세스가 9501 포트를 수신 대기합니다.
    $server = new Swoole\Coroutine\Server('127.0.0.1', 9501, false, true);

    //15번 신호를 수신하면 서버를 종료합니다.
    Process::signal(SIGTERM, function () use ($server) {
        $server->shutdown();
    });

    //새 연결 요청을 받고 자동으로 코루틴을 생성합니다.
    $server->handle(function (Connection $conn) {
        while (true) {
            //데이터를 수신합니다.
            $data = $conn->recv(1);

            if ($data === '' || $data === false) {
                $errCode = swoole_last_error();
                $errMsg = socket_strerror($errCode);
                echo "errCode: {$errCode}, errMsg: {$errMsg}\n";
                $conn->close();
                break;
            }

            //데이터를 전송합니다.
            $conn->send('hello');

            Coroutine::sleep(1);
        }
    });

    //포트를 수신 대기합니다.
    $server->start();
});
$pool->start();
```

!> Cygwin 환경에서 실행하는 경우 단일 프로세스로 변경해야 합니다. `$pool = new Swoole\Process\Pool(1);`
```
