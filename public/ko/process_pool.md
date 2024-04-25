# Swoole\Process\Pool

프로세스 풀은 [Swoole\Server](/server/init)의 Manager로 관리되는 프로세스 모듈을 기반으로합니다. 여러 작업 프로세스를 관리 할 수 있습니다.이 모듈의 핵심 기능은 프로세스 관리입니다. `Process`보다 더 간단하고 높은 수준의 캡슐화를 제공하는 `Process\Pool`은 개발자가 많은 코드를 작성하지 않고도 프로세스 관리 기능을 구현 할 수 있습니다. [Co\Server](/coroutine/server?id=完整示例)와 함께 사용하여, 다중 코어 CPU를 활용할 수있는 순수 코루틴 스타일의 서버 프로그램을 생성할 수 있습니다.
## 프로세스 간 통신

`Swoole\Process\Pool`은 총 세 가지 프로세스 간 통신 방법을 제공합니다:
### 메시지 대기열
`Swoole\Process\Pool->__construct`의 두 번째 매개변수를 `SWOOLE_IPC_MSGQUEUE`로 설정하면 프로세스 간 통신에 메시지 대기열을 사용함을 나타냅니다. `php sysvmsg` 확장을 사용하여 메시지를 전송할 수 있으며, 메시지의 최대 크기는 `65536`을 초과할 수 없습니다.

* **주의사항**

  * `sysvmsg` 확장을 사용하여 메시지를 전송하려면 생성자에 `msgqueue_key`를 반드시 전달해야 합니다.
  * `Swoole`의 하위 수준에서는 `sysvmsg` 확장의 `msg_send` 함수의 두 번째 매개변수 `mtype`을 지원하지 않으므로, 반드시 `0`이 아닌 임의의 값이 전달되어야 합니다.
### 소켓 통신
`Swoole\Process\Pool->__construct`의 두 번째 매개변수를 `SWOOLE_IPC_SOCKET`으로 설정하면 `소켓 통신`을 사용하게 됩니다. 만약 클라이언트와 서버가 같은 기기에 있지 않다면, 이 방법으로 통신할 수 있습니다.

[Swoole\Process\Pool->listen()](/process/process_pool?id=listen) 메서드로 포트를 수신대기하고, [Message 이벤트](/process/process_pool?id=on)를 사용하여 클라이언트가 보낸 데이터를 받아들이며, [Swoole\Process\Pool->write()](/process/process_pool?id=write) 메서드를 사용하여 클라이언트에 대한 응답을 반환합니다.

`Swoole`은 데이터를 이 방식으로 보낼 때, 실제 데이터 앞에 4바이트의 네트워크 바이트 순서의 길이 값을 추가해야 합니다.
```php
$msg = 'Hello Swoole';
$packet = pack('N', strlen($msg)) . $msg;
```
`Swoole\Process\Pool->__construct`의 두 번째 매개변수를 `SWOOLE_IPC_UNIXSOCK`로 설정하면 [UnixSocket](/learn?id=IPC-란)을 사용하는 것을 의미하며, **이 방식을 강력히 권장합니다**.

이 방법은 매우 간단하며, [Swoole\Process\Pool->sendMessage()](/process/process_pool?id=sendMessage) 메서드와 [Message 이벤트](/process/process_pool?id=on)를 통해 프로세스 간 통신을 완료할 수 있습니다.

또는 `코루틴 모드`를 활성화한 후에도 [Swoole\Process\Pool->getProcess()](/process/process_pool?id=getProcess)로 `Swoole\Process` 객체를 얻을 수 있고, [Swoole\Process->exportsocket()](/process/process?id=exportsocket)을 통해 `Swoole\Coroutine\Socket` 객체를 얻어 이를 사용하여 프로세스 간 통신을 구현할 수 있습니다. 그러나 이 경우 [Message 이벤트](/process/process_pool?id=on)를 설정할 수 없습니다.

!> 매개변수 및 환경 설정에 대해 자세히 알아보려면 [생성자](/process/process_pool?id=__construct) 및 [구성 매개변수](/process/process_pool?id=set)를 확인하세요.
## 상수

상수 | 설명
---|---
SWOOLE_IPC_MSGQUEUE | 시스템 [메시지 큐](/learn?id=IPC가 무엇인가요) 통신
SWOOLE_IPC_SOCKET | 소켓 통신
SWOOLE_IPC_UNIXSOCK | [유닉스 소켓](/learn?id=IPC가 무엇인가요) 통신(v4.4+)
## 코루틴 지원

`v4.4.0` 버전에서 코루틴을 지원하게 되었습니다. [Swoole\Process\Pool::__construct](/process/process_pool?id=__construct)를 참조하십시오.
## 사용 예시

```php
use Swoole\Process;
use Swoole\Coroutine;

$pool = new Process\Pool(5);
$pool->set(['enable_coroutine' => true]);
$pool->on('WorkerStart', function (Process\Pool $pool, $workerId) {
    /** 현재는 Worker 프로세스입니다 */
    static $running = true;
    Process::signal(SIGTERM, function () use (&$running) {
        $running = false;
        echo "TERM\n";
    });
    echo("[Worker #{$workerId}] WorkerStart, pid: " . posix_getpid() . "\n");
    while ($running) {
        Coroutine::sleep(1);
        echo "sleep 1\n";
    }
});
$pool->on('WorkerStop', function (\Swoole\Process\Pool $pool, $workerId) {
    echo("[Worker #{$workerId}] WorkerStop\n");
});
$pool->start();
```
## 메서드
### __construct()

构造方法。

```php
Swoole\Process\Pool::__construct(int $worker_num, int $ipc_type = SWOOLE_IPC_NONE, int $msgqueue_key = 0, bool $enable_coroutine = false);
```

* **参数** 

  * **`int $worker_num`**
    * **기능**：작업 프로세스의 수를 지정합니다.
    * **기본값**：없음
    * **다른 값**：없음

  * **`int $ipc_type`**
    * **기능**：프로세스간 통신 모드를 지정합니다.【기본값은`SWOOLE_IPC_NONE`으로 설정되어 있어 아무런 프로세스간 통신 기능을 사용하지 않음을 의미합니다】
    * **기본값**：`SWOOLE_IPC_NONE`
    * **기타 값**：`SWOOLE_IPC_MSGQUEUE`，`SWOOLE_IPC_SOCKET`，`SWOOLE_IPC_UNIXSOCK`

    !> -`SWOOLE_IPC_NONE`으로 설정되어 있을 때`onWorkerStart` 콜백을 설정해야하며, `onWorkerStart`에서 루프 로직을 구현해야 합니다. `onWorkerStart` 함수가 종료되면 작업 프로세스가 즉시 종료되며, 그 이후에는 `Manager` 프로세스가 프로세스를 다시 시작합니다.  
    -`SWOOLE_IPC_MSGQUEUE`으로 설정하면 시스템 메시지 큐 통신을 의미하며 `$msgqueue_key`를 지정하여 메시지 큐의 `KEY`를 지정할 수 있으며, 메시지 큐 KEY가 설정되지 않으면 개인 큐를 할당합니다.  
    -`SWOOLE_IPC_SOCKET`으로 설정하면`소켓`을 사용하여 통신하며, 주소 및 포트를 지정하는 [listen](/process/process_pool?id=listen) 메서드를 사용해야 합니다.  
    -`SWOOLE_IPC_UNIXSOCK`을 사용하면 [unixSocket](/learn?id=IPCとは何ですか)을 사용하여 통신하며, 코루틴 모드에서 사용하며, 프로세스간 통신에 이 방법을 강력히 권장합니다. 구체적인 사용법은 아래 섹션을 참조하세요;  
    -`SWOOLE_IPC_NONE`이 아닌 다른 설정을 사용할 때는`onMessage` 콜백을 설정해야 하며, `onWorkerStart`는 선택 사항이 됩니다.

  * **`int $msgqueue_key`**
    * **기능**：메시지 큐의 `key`
    * **기본값**：`0`
    * **다른 값**：없음

  * **`bool $enable_coroutine`**
    * **기능**：코루틴 지원 여부【코루틴을 사용하면`onMessage` 콜백을 설정할 수 없음】
    * **기본값**：`false`
    * **다른 값**：`true`

* **코루틴 모드**
    
`v4.4.0` 버전부터`Process\Pool` 모듈은 코루틴을 지원하며, 네 번째 인수를`true`로 설정하여 활성화할 수 있습니다. 코루틴을 활성화하면 기본적으로`onWorkerStart`에서 자동으로 코루틴과 [코루틴 컨테이너](/coroutine/scheduler)를 생성하며, 콜백 함수에서 코루틴 관련 `API`를 직접 사용할 수 있습니다. 예를 들어:

```php
$pool = new Swoole\Process\Pool(1, SWOOLE_IPC_NONE, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    while (true) {
        Co::sleep(0.5);
        echo "hello world\n";
    }
});

$pool->start();
```

코루틴을 활성화하면 Swoole은`onMessage` 이벤트 콜백을 설정할 수 없게 하며, 프로세스 간 통신이 필요한 경우 두 번째 인수를`SWOOLE_IPC_UNIXSOCK`로 설정하여 [unixSocket](/learn?id=IPCとは何ですか)을 사용하고, 그런 다음 `$pool->getProcess()->exportSocket()`를 사용하여 [Swoole\Coroutine\Socket](/coroutine_client/socket) 객체를 내보내어`Worker` 프로세스 간 통신을 구현할 수 있습니다. 예를 들어:

 ```php
$pool = new Swoole\Process\Pool(2, SWOOLE_IPC_UNIXSOCK, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    $process = $pool->getProcess(0);
    $socket = $process->exportSocket();
    if ($workerId == 0) {
        echo $socket->recv();
        $socket->send("hello proc1\n");
        echo "proc0 stop\n";
    } else {
        $socket->send("hello proc0\n");
        echo $socket->recv();
        echo "proc1 stop\n";
        $pool->shutdown();
    }
});

$pool->start();
 ```

!> 구체적인 사용법은 [Swoole\Coroutine\Socket](/coroutine_client/socket)와 [Swoole\Process](/process/process?id=exportsocket) 관련 섹션을 참조하세요.

```php
$q = msg_get_queue($key);
foreach (range(1, 100) as $i) {
    $data = json_encode(['data' => base64_encode(random_bytes(1024)), 'id' => uniqid(), 'index' => $i,]);
    msg_send($q, $i, $data, false);
}
```
### set()

설정 매개변수.

```php
Swoole\Process\Pool->set(array $settings): void
```

옵션|타입|설명|기본값
---|---|----|----
enable_coroutine|bool|코루틴 사용 여부를 제어합니다.|false
enable_message_bus|bool|메시지 버스를 활성화합니다. 값이 `true`인 경우, 대량의 데이터를 보낼 때 하부에서 데이터를 작은 조각으로 나누어 상대편에 전송합니다.|false
max_package_size|int|프로세스가 수신할 수 있는 최대 데이터 양을 제한합니다.|2 * 1024 * 1024

* **주의**

  * `enable_message_bus`가 `true`인 경우, `max_package_size`는 작동하지 않습니다. 하부에서 데이터를 작은 조각으로 분할하여 보내고, 수신 데이터도 마찬가지입니다.
  * `SWOOLE_IPC_MSGQUEUE` 모드에서도 `max_package_size`는 작동하지 않습니다. 하부에서 한 번에 최대 `65536`의 데이터 양을 수신합니다.
  * `SWOOLE_IPC_SOCKET` 모드에서는 `enable_message_bus`가 `false`인 경우, 가져온 데이터 양이 `max_package_size`보다 크면 하부에서 연결을 직접 끊습니다.
  * `SWOOLE_IPC_UNIXSOCK` 모드에서는 `enable_message_bus`가 `false`인 경우, 데이터가 `max_package_size`보다 크면 `max_package_size`를 초과하는 데이터가 잘리게 됩니다.
  * 코루틴 모드를 사용 중이고, `enable_message_bus`가 `true`인 경우 `max_package_size`도 작동하지 않습니다. 하부에서 데이터의 분할(전송) 및 병합(수신)을 처리하며, 그렇지 않으면 `max_package_size`에 따라 데이터 양이 제한됩니다.

!> Swoole 버전 >= v4.4.4에서 사용 가능합니다.
### on()

프로세스 풀의 콜백 함수를 설정합니다.

```php
Swoole\Process\Pool->on(string $event, callable $function): bool;
```

* **매개변수** 

  * **`string $event`**
    * **기능**：이벤트를 지정합니다.
    * **기본값**：없음
    * **다른 값**：없음

  * **`callable $function`**
    * **기능**：콜백 함수를 지정합니다.
    * **기본값**：없음
    * **다른 값**：없음

* **이벤트**

  * **onWorkerStart** 워커 프로세스 시작

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Pool 객체
  * @param int $workerId   현재 작업 중인 워커 프로세스의 번호, 하위 수준에서 자식 프로세스를 번호 매깁니다.
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStart', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} 가 시작되었습니다.\n";
  });
  ```

  * **onWorkerStop** 워커 프로세스 종료

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Pool 객체
  * @param int $workerId   현재 작업 중인 워커 프로세스의 번호, 하위 수준에서 자식 프로세스를 번호 매깁니다.
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStop', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} 가 중지되었습니다.\n";
  });
  ```

  * **onMessage** 메시지 수신

  !> 외부로부터 전달된 메시지를 수신합니다. 한 번의 연결에 대해 한 번의 메시지만 전달할 수 있으며 `PHP-FPM`의 단일 연결 메커니즘과 유사합니다.

  ```php
  /**
    * @param \Swoole\Process\Pool $pool Pool 객체
    * @param string $data 메시지 데이터 내용
   */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('Message', function(Swoole\Process\Pool $pool, string $data){
    var_dump($data);
  });
  ```

  !> 이벤트 이름은 대소문자를 구분하지 않습니다. `WorkerStart`, `workerStart` 또는 `workerstart`은 모두 동일합니다.
### listen()

`SOCKET`을 듣습니다. `$ipc_mode = SWOOLE_IPC_SOCKET` 일 때만 사용할 수 있습니다.

```php
Swoole\Process\Pool->listen(string $host, int $port = 0, int $backlog = 2048): bool
```

* **매개변수** 

  * **`string $host`**
    * **기능**: 리스닝할 주소【`TCP` 및 [unixSocket](/learn?id=什게이ipc) 두 가지 유형을 지원합니다. `127.0.0.1`은 `TCP` 주소를 듣는 것을 의미하며, `$port`를 지정해야 합니다. `unix:/tmp/php.sock`은 [unixSocket](/learn?id=ipc이 무엇인가요?) 주소를 듣는 것을 의미합니다】
    * **기본값**: 없음
    * **다른 값**: 없음

  * **`int $port`**
    * **기능**: 리스닝할 포트【`TCP` 모드에서 지정해야 함】
    * **기본값**: `0`
    * **다른 값**: 없음

  * **`int $backlog`**
    * **기능**: 들을 수 있는 대기열 길이
    * **기본값**: `2048`
    * **다른 값**: 없음

* **반환값**

  * 성공적인 듣기는 `true`를 반환합니다.
  * 듣기 실패 시 `false`를 반환하며, 오류 코드를 얻으려면 `swoole_errno`을 호출할 수 있습니다. 듣기 실패 시, `start`를 호출하면 즉시 `false`가 반환됩니다.

* **통신 프로토콜**

    데이터를 듣는 포트로 보낼 때, 클라이언트는 요청 앞에 4바이트의 네트워크 바이트 오더의 길이 값을 추가해야 합니다. 프로토콜 형식은 다음과 같습니다:

```php
// $msg 보낼 데이터
$packet = pack('N', strlen($msg)) . $msg;
```

* **사용 예시**

```php
$pool->listen('127.0.0.1', 8089);
$pool->listen('unix:/tmp/php.sock');
```
### write()

상대에게 데이터를 쓰려면 `$ipc_mode`이 `SWOOLE_IPC_SOCKET`일 때만 사용할 수 있습니다.

```php
Swoole\Process\Pool->write(string $data): bool
```

!> 이 메서드는 메모리 조작이며 `IO` 소비가 없습니다. 데이터를 보내는 작업은 동기화 블로킹 `IO`입니다.

* **매개변수** 

  * **`string $data`**
    * **기능**：쓰기 위한 데이터 내용【`write`를 여러 번 호출할 수 있으며, `onMessage` 함수가 종료된 후에 모든 데이터를 `socket`에 쓰고 연결을 `close`합니다】
    * **기본값**：없음
    * **다른 값**：없음

* **사용 예시**

  * **서버**

    ```php
    $pool = new Swoole\Process\Pool(2, SWOOLE_IPC_SOCKET);
    
    $pool->on("Message", function ($pool, $message) {
        echo "Message: {$message}\n";
        $pool->write("hello ");
        $pool->write("world ");
        $pool->write("\n");
    });
    
    $pool->listen('127.0.0.1', 8089);
    $pool->start();
    ```

  * **호출하는 쪽**

    ```php
    $fp = stream_socket_client("tcp://127.0.0.1:8089", $errno, $errstr) or die("error: $errstr\n");
    $msg = json_encode(['data' => 'hello', 'uid' => 1991]);
    fwrite($fp, pack('N', strlen($msg)) . $msg);
    sleep(1);
    // will display hello world\n
    $data = fread($fp, 8192);
    var_dump(substr($data, 4, unpack('N', substr($data, 0, 4))[1]));
    fclose($fp);
    ```  
### sendMessage()

타겟 프로세스로 데이터를 전송하려면 `$ipc_mode`이 `SWOOLE_IPC_UNIXSOCK`일 때만 사용할 수 있습니다.

```php
Swoole\Process\Pool->sendMessage(string $data, int $dst_worker_id): bool
```

* **Parameters** 

  * **`string $data`**
    * **Description**: 전송할 데이터
    * **Default**: 없음
    * **Other values**: 없음

  * **`int $dst_worker_id`**
    * **Description**: 타겟 프로세스 ID
    * **Default**: `0`
    * **Other values**: 없음

* **Return Value**

  * 성공 시 `true` 반환
  * 실패 시 `false` 반환

* **주의사항**

  * 데이터가 `max_package_size`보다 크고 `enable_message_bus`가 `false`인 경우, 타겟 프로세스는 데이터를 수신할 때 데이터를 잘라냅니다.

```php
<?php
use Swoole\Process;
use Swoole\Coroutine;

$pool = new Process\Pool(2, SWOOLE_IPC_UNIXSOCK);
$pool->set(['enable_coroutine' => true, 'enable_message_bus' => false, 'max_package_size' => 2 * 1024]);

$pool->on('WorkerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    if ($workerId == 0) {
        $pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();

// int(2048)


$pool = new Process\Pool(2, SWOOLE_IPC_UNIXSOCK);
$pool->set(['enable_coroutine' => true, 'enable_message_bus' => true, 'max_package_size' => 2 * 1024]);

$pool->on('WorkerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    if ($workerId == 0) {
        $pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();

// int(6000)
```
### start()

작업 프로세스를 시작합니다.

```php
Swoole\Process\Pool->start(): bool
```

!> 시작에 성공하면 현재 프로세스는 `wait` 상태로 들어가며 작업 프로세스를 관리합니다;  
시작에 실패하면 `false`를 반환하며 `swoole_errno`을 사용하여 오류 코드를 얻을 수 있습니다.

* **사용 예시**

```php
$workerNum = 10;
$pool = new Swoole\Process\Pool($workerNum);

$pool->on("WorkerStart", function ($pool, $workerId) {
    echo "Worker#{$workerId} is started\n";
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);
    $key = "key1";
    while (true) {
         $msg = $redis->brpop($key, 2);
         if ( $msg == null) continue;
         var_dump($msg);
     }
});

$pool->on("WorkerStop", function ($pool, $workerId) {
    echo "Worker#{$workerId} is stopped\n";
});

$pool->start();
```

* **프로세스 관리**

  * 어떤 작업 프로세스가 치명적인 오류를 만나거나 자체적으로 종료되면 관리자가 회수하여 좀비 프로세스를 방지합니다
  * 작업 프로세스가 종료되면 관리자가 자동으로 새 작업 프로세스를 실행합니다
  * 주 프로세스가 `SIGTERM` 신호를받으면`fork` 새로운 프로세스를 중지하고 실행중인 모든 작업 프로세스를`kill`합니다
  * 주 프로세스가 `SIGUSR1` 신호를받으면 실행중인 모든 작업 프로세스를 차례로`kill`하고 새 작업 프로세스를 다시 시작합니다

* **신호 처리**

  밑바닥에서는 주 프로세스 (매니저 프로세스)의 신호 처리만 설정되어 있으며`Worker` 작업 프로세스에 신호를 설정하지 않았으므로 개발자가 직접 신호를 듣습니다.

  - 작업 프로세스가 비동기 모드인 경우 [Swoole\Process::signal](/process/process?id=signal)를 사용하여 신호를 감시합니다
  - 작업 프로세스가 동기 모드인 경우`pcntl_signal` 및`pcntl_signal_dispatch`를 사용하여 신호를 감시합니다

  작업 프로세스는`SIGTERM` 신호를 듣도록 설정해야 하며, 주 프로세스가이 프로세스를 종료해야 할 경우에는`SIGTERM` 신호를 보냅니다. 작업 프로세스가`SIGTERM` 신호를 듣지 않았으면 밑바닥은 현재 프로세스를 강제로 종료시켜 일부 로직 손실을 초래할 수 있습니다.

```php
$pool->on("WorkerStart", function ($pool, $workerId) {
    $running = true;
    pcntl_signal(SIGTERM, function () use (&$running) {
        $running = false;
    });
    echo "Worker#{$workerId} is started\n";
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);
    $key = "key1";
    while ($running) {
         $msg = $redis->brpop($key);
         pcntl_signal_dispatch();
         if ( $msg == null) continue;
         var_dump($msg);
     }
});
```
### stop()

현재 프로세스 소켓을 이벤트 루프에서 제거하고, 코루틴을 시작한 후에 이 함수가 작동합니다.

```php
Swoole\Process\Pool->stop(): bool
```  
### shutdown()

작업 프로세스를 종료합니다.

```php
Swoole\Process\Pool->shutdown(): bool
```
### getProcess()

현재 작업 프로세스 객체를 가져옵니다. [Swoole\Process](/process/process) 객체를 반환합니다.

!> Swoole 버전 >= `v4.2.0`에서 사용 가능

```php
Swoole\Process\Pool->getProcess(int $worker_id): Swoole\Process
```

* **매개변수** 

  * **`int $worker_id`**
    * **기능**：`worker`를 지정하여 가져옴 【선택 사항, 기본값은 현재 `worker`】
    * **기본값**：없음
    * **다른 값**：없음

!> 반드시 `start` 이후에, 작업 프로세스의 `onWorkerStart` 또는 다른 콜백 함수 내에서 호출해야 합니다.  
반환되는 `Process` 객체는 싱글톤 모드이며, 작업 프로세스에서 `getProcess()`를 반복 호출하면 동일한 객체가 반환됩니다.

* **사용 예시**

```php
$pool = new Swoole\Process\Pool(3);

$pool->on('WorkerStart', function ($pool, $workerId) {
    $process = $pool->getProcess();
    $process->exec('/usr/local/bin/php', ['-r', 'var_dump(swoole_version());']);
});

$pool->start();
```
### detach()

현재 Worker 프로세스를 관리에서 분리하고, 하위에서 즉시 새로운 프로세스를 만듭니다. 이전 프로세스는 더 이상 데이터를 처리하지 않고, 애플리케이션 코드에서 수명주기를 직접 관리해야 합니다.

!> Swoole 버전 >= `v4.7.0`에서 사용 가능

```php
Swoole\Process\Pool->detach(): bool
```
