# Swoole\Process

Swoole提供的进程管理模块，用来替代PHP的`pcntl`  

!> 此模块比较底层，是操作系统进程管理的封装，使用者需要具备`Linux`系统多进程编程经验。

`PHP`自带的`pcntl`，存在很多不足，如：

* 没有提供进程间通信的功能
* 不支持重定向标准输入和输出
* 只提供了`fork`这样原始的接口，容易使用错误

`Process`提供了比`pcntl`更强大的功能，更易用的`API`，使PHP在多进程编程方面更加轻松。

`Process`提供了如下特性：

* 可以方便的实现进程间通讯
* 支持重定向标准输入和输出，在子进程内`echo`不会打印屏幕，而是写入管道，读键盘输入可以重定向为管道读取数据
* 提供了[exec](/process/process?id=exec)接口，创建的进程可以执行其他程序，与原`PHP`父进程之间可以方便的通信
* 在协程环境中无法使用`Process`模块，可以使用`runtime hook`+`proc_open`实现，参考[协程进程管理](/coroutine/proc_open)
```php
use Swoole\Process;

for ($n = 1; $n <= 3; $n++) {
    $process = new Process(function () use ($n) {
        echo '자식 #' . getmypid() . "가 시작되었고 {$n}초 동안 잠듭니다" . PHP_EOL;
        sleep($n);
        echo '자식 #' . getmypid() . '가 종료되었습니다' . PHP_EOL;
    });
    $process->start();
}
for ($n = 3; $n--;) {
    $status = Process::wait(true);
    echo "#{$status['pid']}을 재활용했습니다, 코드={$status['code']}, 시그널={$status['signal']}" . PHP_EOL;
}
echo '부모 #' . getmypid() . '가 종료되었습니다' . PHP_EOL;
```
## 속성
### pipe

[unixSocket](/learn?id=什么是IPC)의 파일 기술자입니다.

```php
public int $pipe;
``` 
### msgQueueId

`id` 메시지 대기열입니다.

```php
public int $msgQueueId;
```
### msgQueueKey

메시지 대기열의 `key`。

```php
public string $msgQueueKey;
```
### pid

현재 프로세스의 `pid`.

```php
public int $pid;
```
### id

현재 프로세스 `id`.

```php
public int $id;
```
## 상수
매개변수 | 기능
---|---
Swoole\Process::IPC_NOWAIT | 메시지 큐에 데이터가 없을 때 즉시 반환
Swoole\Process::PIPE_READ | 읽기 소켓을 닫음
Swoole\Process::PIPE_WRITE | 쓰기 소켓을 닫음
```python
def hello():
    return "안녕하세요"
```

번역:
```python
def hello():
    return "Hello"
```
### __construct()

생성자 메서드.

```php
Swoole\Process->__construct(callable $function, bool $redirect_stdin_stdout = false, int $pipe_type = SOCK_DGRAM, bool $enable_coroutine = false)
```

* **매개변수** 

  * **`callable $function`**
    * **기능**：자식 프로세스가 생성된 후 실행되어야 하는 함수【하위 수준에서 함수는 객체의`callback` 속성에 자동으로 저장됨】, 주의: 이 속성은 `private`으로 클래스에만 사용됨.
    * **기본값**：없음
    * **다른 값**：없음

  * **`bool $redirect_stdin_stdout`**
    * **기능**：자식 프로세스의 표준 입력 및 출력을 재지정합니다.【이 옵션을 활성화하면 자식 프로세스 내에서 출력된 내용이 화면에 표시되지 않고 대신 메인 프로세스 파이프에 쓰여집니다. 키보드 입력은 파이프에서 데이터를 읽는 것으로 바뀝니다. 기본적으로 블로킹 읽기입니다. [exec()](/process/process?id=exec) 메서드 내용 참조】
    * **기본값**：없음
    * **다른 값**：없음

  * **`int $pipe_type`**
    * **기능**：[unixSocket](/learn?id=什么是IPC) 유형【`$redirect_stdin_stdout`를 활성화한 후 이 옵션은 사용자 매개변수를 무시하고 강제로`SOCK_STREAM`으로 설정됩니다. 자식 프로세스 내에서 프로세스 간 통신이 없으면 `0`으로 설정할 수 있습니다】
    * **기본값**：`SOCK_DGRAM`
    * **다른 값**：`0`、`SOCK_STREAM`

  * **`bool $enable_coroutine`**
    * **기능**：`callback function`에서 코루틴을 활성화하며, 활성화 후에는 자식 프로세스 함수에서 코루틴 API를 직접 사용할 수 있습니다.
    * **기본값**：`false`
    * **다른 값**：`true`
    * **버전 영향**：Swoole 버전 >= v4.3.0

* **[unixSocket](/learn?id=什么是IPC) 유형**

unixSocket 유형 | 설명
---|---
0 | 생성 안 함
1 | [SOCK_STREAM](/learn?id=IPC란 무엇인가?) 유형의 unixSocket 생성
2 | [SOCK_DGRAM](/learn?id=IPC란 무엇인가?) 유형의 unixSocket 생성
### useQueue()

프로세스 간 통신을 위해 메시지 대기열을 사용합니다.

```php
Swoole\Process->useQueue(int $key = 0, int $mode = SWOOLE_MSGQUEUE_BALANCE, int $capacity = -1): bool
```

* **매개변수**

  * **`int $key`**
    * **기능**: 메시지 대기열의 키. 0보다 작거나 같은 값이 전달되면, 기본적으로 `ftok` 함수를 사용하여 현재 실행 파일의 파일 이름을 매개 변수로 사용하여 해당 키를 생성합니다.
    * **기본값**: `0`
    * **기타 값**: 없음

  * **`int $mode`**
    * **기능**: 프로세스 간 통신 모드입니다.
    * **기본값**: `SWOOLE_MSGQUEUE_BALANCE`. `Swoole\Process::pop()`는 대기열 첫 번째 메시지를 반환하며, `Swoole\Process::push()`는 메시지에 특정 유형을 추가하지 않습니다.
    * **기타 값**: `SWOOLE_MSGQUEUE_ORIENT`. `Swoole\Process::pop()`은 대기열에서 `프로세스 id + 1` 특정 데이터 유형의 메시지를 반환하고, `Swoole\Process::push()`는 메시지에 `프로세스 id + 1`의 유형을 추가합니다.

  * **`int $capacity`**
    * **기능**: 메시지 대기열에 저장할 수 있는 최대 메시지 수입니다.
    * **기본값**: `-1`
    * **기타 값**: 없음

* **주의사항**

  * 메시지 대기열에 데이터가 없는 경우, `Swoole\Porcess->pop()`은 계속 차단되거나 새 데이터를 수용할 공간이 없으면 `Swoole\Porcess->push()`도 계속 차단됩니다. 차단을 원치 않으면, `mode` 값은 `SWOOLE_MSGQUEUE_BALANCE|Swoole\Process::IPC_NOWAIT` 또는 `SWOOLE_MSGQUEUE_ORIENT|Swoole\Process::IPC_NOWAIT` 여야 합니다.
### statQueue()

메시지 큐 상태를 가져옵니다.

```php
Swoole\Process->statQueue(): array|false
```

* **리턴 값** 

  * 성공 시 배열이 반환됩니다. 배열은 두 개의 키-값 쌍으로 구성되어 있으며, `queue_num`은 현재 큐에있는 메시지의 총 수를 나타내고, `queue_bytes`는 현재 큐의 메시지 총 크기를 나타냅니다.
  * 실패할 경우 `false`가 반환됩니다.
### freeQueue()

메시지 큐를 파괴합니다.

```php
Swoole\Process->freeQueue(): bool
```

* **반환 값**

  * 성공하면 `true`가 반환됩니다.
  * 실패하면 `false`가 반환됩니다.
### pop()

데이터를 메시지 큐에서 가져옵니다.

```php
Swoole\Process->pop(int $size = 65536): string|false
```

* **파라미터** 

  * **`int $size`**
    * **기능**：가져올 데이터의 크기.
    * **기본 값**：`65536`
    * **다른 값**：`없음`

* **반환 값** 

  * 성공 시 `string`을 반환합니다.
  * 실패 시 `false`를 반환합니다.

* **주의사항**

  * 메시지 큐 유형이 `SW_MSGQUEUE_BALANCE` 인 경우, 큐의 첫 번째 메시지를 반환합니다.
  * 메시지 큐 유형이 `SW_MSGQUEUE_ORIENT` 인 경우, 큐에서 현재 프로세스 id + 1 유형의 첫 번째 메시지를 반환합니다.
### push()

데이터를 메시지 대기열로 보냅니다.

```php
Swoole\Process->push(string $data): bool
```

* **매개변수** 

  * **`string $data`**
    * **기능**：전송할 데이터.
    * **기본값**：``
    * **다른 값**：`없음`


* **반환 값** 

  * `true`가 반환되면 성공입니다.
  * 실패 시 `false`가 반환됩니다.

* **주의사항**

  * 메시지 대기열 유형이 `SW_MSGQUEUE_BALANCE` 인 경우 데이터는 메시지 대기열에 직접 삽입됩니다.
  * 메시지 대기열 유형이 `SW_MSGQUEUE_ORIENT` 인 경우 데이터에 현재 `프로세스 id + 1` 유형이 추가됩니다.
### setTimeout()

메시지 큐의 읽기 및 쓰기 시간을 설정합니다.

```php
Swoole\Process->setTimeout(float $seconds): bool
```

* **Parameters**

  * **`float $seconds`**
    * **Description**: Timeout period
    * **Default**: `None`
    * **Other values**: `None`

* **Return Value**

  * Returns `true` on success.
  * Returns `false` on failure.
### setBlocking()

설정 메시지 큐 소켓이 차단되는지 여부.

```php
Swoole\Process->setBlocking(bool $$blocking): void
```

* **매개변수** 

  * **`bool $blocking`**
    * **기능**：차단 여부, `true`는 차단, `false`는 비차단
    * **기본값**：`없음`
    * **다른 값**：`없음`

* **주의**

  * 새로 생성된 프로세스 소켓은 기본적으로 차단되어 있으므로, UNIX 도메인 소켓 통신시 메시지를 보내거나 읽을 때 프로세스가 차단됩니다.
### write()

부모 자식 프로세스간의 메시지 쓰기(UNIX 도메인 소켓).

```php
Swoole\Process->write(string $data): false|int
```

* **매개변수** 

  * **`string $data`**
    * **기능**：쓸 데이터
    * **기본값**：`없음`
    * **다른 값**：`없음`


* **반환값** 

  * 성공 시 `int`를 반환하며, 쓴 바이트 수를 나타냅니다.
  * 실패 시 `false`가 반환됩니다.
### read()

부모 자식 프로세스 간의 메시지 수신 (UNIX 도메인 소켓).

```php
Swoole\Process->read(int $size = 8192): false|string
```

* **Parameters** 

  * **`int $size`**
    * **Description**: 읽어야 하는 데이터의 크기
    * **Default**: `8192`
    * **Other values**: 없음


* **Return Value** 

  * 성공시 `string` 반환.
  * 실패시 `false` 반환.
### set()

설정 매개변수입니다.

```php
Swoole\Process->set(array $settings): void
```

`enable_coroutine`을 사용하여 코루틴을 활성화할지 여부를 제어할 수 있습니다. 이는 생성자의 네 번째 매개변수와 동일한 역할을 합니다.

```php
Swoole\Process->set(['enable_coroutine' => true]);
```

!> Swoole 버전 >= v4.4.4에서 사용 가능합니다.
### start()

`fork`시스템 콜을 실행하여 자식 프로세스를 시작합니다. `Linux` 시스템에서 프로세스를 생성하는 데는 수백 마이크로초가 걸립니다.

```php
Swoole\Process->start(): int|false
```

* **반환값**

  * 성공하면 자식 프로세스의`PID`를 반환합니다.
  * 실패하면 `false`를 반환합니다. [swoole_errno](/functions?id=swoole_errno) 및 [swoole_strerror](/functions?id=swoole_strerror)을 사용하여 오류 코드 및 오류 메시지를 얻을 수 있습니다.

* **주의사항**

  * 자식 프로세스는 부모 프로세스의 메모리 및 파일 핸들을 상속합니다.
  * 자식 프로세스가 시작될 때 부모 프로세스로부터 상속된 [EventLoop](/learn?id=이벤트루프)、[Signal](/process/process?id=신호) 및 [Timer](/timer)가 지워집니다.
  
  !> 자식 프로세스 실행 후에는 부모 프로세스의 메모리 및 리소스를 유지하며, 부모 프로세스에서 redis 연결을 만들었을 경우 자식 프로세스에서도 이 객체가 유지되어 모든 작업이 동일한 연결을 대상으로 합니다. 아래 예시를 참고하세요.

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);

function callback_function() {
    swoole_timer_after(1000, function () {
        echo "hello world\n";
    });
    global $redis;//같은 연결
};

swoole_timer_tick(1000, function () {
    echo "부모 타이머\n";
});//상속되지 않음

Swoole\Process::signal(SIGCHLD, function ($sig) {
    while ($ret = Swoole\Process::wait(false)) {
        // 자식 프로세스 생성
        $p = new Swoole\Process('callback_function');
        $p->start();
    }
});

// 자식 프로세스 생성
$p = new Swoole\Process('callback_function');

$p->start();
```

!> 1. 자식 프로세스가 시작되면 부모 프로세스에서 생성한[Swoole\Timer::tick](/timer?id=tick) 의 생성된 타이머, [Process::signal](/process/process?id=signal)이 수신한 신호 및 [Swoole\Event::add](/event?id=add)로 추가된 이벤트 리스너가 자동으로 제거됩니다.
2. 자식 프로세스는 부모 프로세스에서 생성한 `$redis` 연결 객체를 상속받으며, 부모 및 자식 프로세스가 동일한 연결을 사용합니다.
### exportSocket()

`unixSocket`을 `Swoole\Coroutine\Socket` 객체로 내보내어 다른 프로세스 간 통신에 사용할 수 있습니다. 자세한 사용법은 [Coroutine\socket](/coroutine_client/socket) 및 [IPC 통신](/learn?id=IPC가 무엇인가요?)을 참조하세요.

```php
Swoole\Process->exportSocket(): Swoole\Coroutine\Socket|false
```

!> 이 메서드를 여러 번 호출하더라도 동일한 객체가 반환됩니다.  
`exportSocket()`으로 내보낸 `socket`은 새로운 `fd`이며 이를 닫아도 원래 프로세스의 파이프에는 영향을 주지 않습니다.  
`Swoole\Coroutine\Socket` 객체이므로 [코루틴 스케줄러](/coroutine/scheduler) 내에서만 사용할 수 있습니다. 따라서 `Swoole\Process` 생성자의 `$enable_coroutine` 매개변수는 true이어야 합니다.  
부모 프로세스도 `Swoole\Coroutine\Socket` 객체를 사용하려면 수동으로 `Coroutine\run()`을 호출하여 코루틴 컨테이너를 만들어야 합니다.

* **반환값**

  * 성공 시 `Coroutine\Socket` 객체 반환
  * 프로세스가 unixSocket을 생성하지 않은 경우 작업 실패 시 `false` 반환

* **사용 예시**

간단한 부모-자식 프로세스 통신 구현:

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    echo $socket->recv();
    $socket->send("hello master\n");
    echo "proc1 stop\n";
}, false, 1, true);

$proc1->start();

// 부모 프로세스에서 코루틴 컨테이너 생성
run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    $socket->send("hello pro1\n");
    var_dump($socket->recv());
});
Process::wait(true);
```

더 복잡한 통신 예제:

```php
use Swoole\Process;
use Swoole\Timer;
use function Swoole\Coroutine\run;

$process = new Process(function ($proc) {
    Timer::tick(1000, function () use ($proc) {
        $socket = $proc->exportSocket();
        $socket->send("hello master\n");
        echo "child timer\n";
    });
}, false, 1, true);

$process->start();

run(function() use ($process) {
    Process::signal(SIGCHLD, static function ($sig) {
        while ($ret = Swoole\Process::wait(false)) {
            /* clean up then event loop will exit */
            Process::signal(SIGCHLD, null);
            Timer::clearAll();
        }
    });
    /* here you can run your other async or coroutine code */
    Timer::tick(500, function () {
        echo "parent timer\n";
    });

    $socket = $process->exportSocket();
    while (1) {
        var_dump($socket->recv());
    }
});
```

!> 기본 유형은 `SOCK_STREAM`이며 TCP 데이터 패킷 경계 문제를 처리해야 합니다. [Coroutine\socket](/coroutine_client/socket)의 `setProtocol()` 메서드를 참조하세요.

`SOCK_DGRAM` 유형을 사용하여 IPC 통신을 수행하면 TCP 데이터 패킷 경계 문제를 처리할 필요가 없습니다. [IPC 통신](/learn?id=IPC가 무엇인가요?)을 참조하세요:

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

//IPC 통신에서는 SOCK_DGRAM 유형의 소켓에는 sendto / recvfrom 함수를 사용하지 않아도 send/recv가 가능합니다.
$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    while (1) {
        var_dump($socket->send("hello master\n"));
    }
    echo "proc1 stop\n";
}, false, 2, 1); // 생성자 파이프 유형을 2로 지정하여 SOCK_DGRAM으로 설정

$proc1->start();

run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    Swoole\Coroutine::sleep(5);
    var_dump(strlen($socket->recv())); // 한 번의 recv로 "hello master\n" 문자열 하나만 받게 됩니다. 여러 개의 "hello master\n" 문자열이 수신되지 않습니다.
});

Process::wait(true);
```
### name()

프로세스 이름을 변경합니다. 이 함수는 [swoole_set_process_name](/functions?id=swoole_set_process_name)의 별칭입니다.

```php
Swoole\Process->name(string $name): bool
```

!> `exec`을 실행한 후에는 새로운 프로그램이 프로세스 이름을 재설정할 것입니다. `name` 메서드는 `start` 이후의 자식 프로세스 콜백 함수 내에서 사용되어야 합니다.
### exec()

`exec()` 함수는 외부 프로그램을 실행하며, 이 함수는 `exec` 시스템 호출의 래퍼입니다.

```php
Swoole\Process->exec(string $execfile, array $args);
```

* **매개변수**

  * **`string $execfile`**
    * **기능** : 실행할 파일의 절대 경로를 지정합니다. 예: `"/usr/bin/python"`
    * **기본값** : 없음
    * **기타값** : 없음

  * **`array $args`**
    * **기능** : `exec`의 매개변수 목록  (예: `array('test.py', 123)`은 `python test.py 123`과 동일)
    * **기본값** : 없음
    * **기타값** : 없음

성공적으로 실행되면, 현재 프로세스의 코드 영역이 새 프로그램으로 대체됩니다. 자식 프로세스가 다른 프로그램으로 변화합니다. 부모 프로세스와 현재 프로세스는 여전히 부모-자식 프로세스 관계를 유지합니다.

부모 프로세스와 새로운 프로세스 간에는 표준 입력 출력을 통해 통신할 수 있으며, 표준 입력 출력 리다이렉션을 활성화해야 합니다.

!> `$execfile`은 절대 경로를 사용해야 하며, 그렇지 않으면 파일이 존재하지 않는 오류가 발생합니다.  
`exec` 시스템 호출은 지정된 프로그램으로 현재 프로그램을 덮어쓸 것이므로, 자식 프로세스는 표준 출력을 읽고 부모 프로세스와 통신해야 합니다.  
`redirect_stdin_stdout = true`를 지정하지 않은 경우, `exec`를 실행한 후 자식 프로세스와 부모 프로세스는 통신할 수 없습니다.

* **사용 예시**

예 1: `Swoole\Process`에서 [Swoole\Server](/server/init)를 사용할 수 있지만, 보안을 위해 `$process->start` 후에 `$worker->exec()`을 호출해야 합니다. 코드는 다음과 같습니다.

```php
$process = new Swoole\Process('callback_function', true);

$pid = $process->start();

function callback_function(Swoole\Process $worker)
{
    $worker->exec('/usr/local/bin/php', array(__DIR__.'/swoole_server.php'));
}

Swoole\Process::wait();
```

예 2: Yii 프로그램 시작

```php
$process = new \Swoole\Process(function (\Swoole\Process $childProcess) {
    // 이러한 방식은 지원되지 않음
    // $childProcess->exec('/usr/local/bin/php /var/www/project/yii-best-practice/cli/yii t/index -m=123 abc xyz');

    // exec 시스템 호출 래핑
    // 절대 경로
    // 매개변수를 배열에 분리해서 넣어야 합니다.
    $childProcess->exec('/usr/local/bin/php', ['/var/www/project/yii-best-practice/cli/yii', 't/index', '-m=123', 'abc', 'xyz']); // exec 시스템 호출
});
$process->start(); // 자식 프로세스 실행
```

예 3: 부모 프로세스와 `exec` 자식 프로세스가 표준 입력 출력을 사용하여 통신하는 예시:

```php
// exec - exec 프로세스와 통신
use Swoole\Process;
use function Swoole\Coroutine\run;

$process = new Process(function (Process $worker) {
    $worker->exec('/bin/echo', ['hello']);
}, true, 1, true); // 표준 입력 출력 리다이렉션 활성화 필요

$process->start();

run(function() use($process) {
    $socket = $process->exportSocket();
    echo "from exec: " . $socket->recv() . "\n";
});
```

예 4: 셸 명령어 실행

`exec` 메서드는 `PHP`의 `shell_exec`와 다르며, 더 낮은 수준의 시스템 호출 래핑입니다. 셸 명령을 실행해야 하는 경우, 다음 방법을 사용하세요:

```php
$worker->exec('/bin/sh', array('-c', "cp -rf /data/test/* /tmp/test/"));
```
### close()

`close()`은 생성한 [unixSocket](/learn?id=IPC이란 무엇인가)을 닫을 때 사용됩니다.

```php
Swoole\Process->close(int $which): bool
```

* **매개변수** 

  * **`int $which`**
    * **기능**: unixSocket은 양방향 통신이므로, 어느 쪽을 닫을지 지정합니다. 【기본값으로 `0`은 읽기 및 쓰기 모두 닫음, `1`: 쓰기 닫힘, `2`: 읽기 닫힘】
    * **기본값**: `0`, 읽기 및 쓰기 소켓을 닫음.
    * **다른 값**: `Swoole/Process::SW_PIPE_CLOSE_READ`는 읽기 소켓을 닫고, `Swoole/Process::SW_PIPE_CLOSE_WRITE`는 쓰기 소켓을 닫습니다.

!> 특정 상황에서 `Process` 객체를 해제할 수 없는 경우가 있으며, 프로세스를 계속 생성하면 연결이 누출될 수 있습니다. 이 함수를 호출하여 `unixSocket`을 직접 닫고 리소스를 해제할 수 있습니다.
### exit()

부모 프로세스를 종료합니다.

```php
Swoole\Process->exit(int $status = 0);
```

* **매개변수** 

  * **`int $status`**
    * **기능**：프로세스의 상태 코드로 종료합니다【0이면 정상 종료하며 정리 작업이 계속 실행됩니다】
    * **기본 값**：`0`
    * **다른 값**：없음

!> 정리 작업에는 다음이 포함됩니다:

  * `PHP`의 `shutdown_function`
  * 객체 소멸(`__destruct`)
  * 다른 확장의 `RSHUTDOWN` 함수

`$status`가 `0`이 아닌 경우 비정상 종료이므로 즉시 프로세스를 중지하고 관련 프로세스 종료 정리 작업을 더 이상 수행하지 않습니다.

부모 프로세스에서 `Process::wait`를 실행하여 자식 프로세스 종료 이벤트와 상태 코드를 얻을 수 있습니다.
### kill()

지정된 `pid` 프로세스에 신호를 보냅니다.

```php
Swoole\Process::kill(int $pid, int $signo = SIGTERM): bool
```

* **매개변수**

  * **`int $pid`**
    * **기능**：프로세스 `pid`
    * **기본값**：없음
    * **다른 값**：없음

  * **`int $signo`**
    * **기능**：보내는 신호【`$signo=0`일 경우, 프로세스의 존재여부를 확인하며, 신호는 보내지 않음】
    * **기본값**：`SIGTERM`
    * **다른 값**：없음
### signal()

비동기 신호 리스너를 설정합니다.

```php
Swoole\Process::signal(int $signo, callable $callback): bool
```

이 메서드는 `signalfd`와 [EventLoop](/learn?id=이벤트 루프란 무엇인가)에 기반하여 비동기 `IO`를 제공하며, 블로킹 프로그램에서 사용할 수 없습니다. 등록된 리스너 콜백 함수가 스케줄링되지 않도록 할 수 있습니다.

동기 블로킹 프로그램에서는 `pcntl_signal`을 제공하는 `pcntl` 확장을 사용할 수 있습니다.

이 신호의 콜백 함수가 이미 설정되어 있는 경우, 다시 설정하면 이전 설정이 덮어씁니다.

* **매개변수** 

  * **`int $signo`**
    * **목적**：신호
    * **기본값**: 없음
    * **기타 값**: 없음

  * **`callable $callback`**
    * **목적**：콜백 함수【`$callback`이 `null`이면 신호 리스닝이 제거됨】
    * **기본값**: 없음
    * **기타 값**: 없음

!> [Swoole\Server](/server/init)에서 `SIGTERM` 및 `SIGALAM`과 같은 특정 신호 리스너를 설정할 수 없습니다.

* **사용 예시**

```php
Swoole\Process::signal(SIGTERM, function($signo) {
     echo "shutdown.";
});
```

!> `v4.4.0` 버전에서는 프로세스의 [EventLoop](/learn?id=이벤트 루프란 무엇인가)에 신호 리스너 이외의 이벤트가 없는 경우(예: 타이머 등), 프로세스가 직접 종료됩니다.

```php
Swoole\Process::signal(SIGTERM, function($signo) {
     echo "shutdown.";
});
Swoole\Event::wait();
```

상기 프로그램은 [EventLoop](/learn?id=이벤트 루프란 무엇인가)로 진입하지 않으며, `Swoole\Event::wait()`는 즉시 반환되어 프로세스가 종료됩니다.
### wait()

종료된 하위 프로세스를 회수합니다.

!> `v4.5.0` 이상의 Swoole 버전에서는 코루틴 버전의 `wait()`를 사용하는 것이 좋습니다. [Swoole\Coroutine\System::wait()](/coroutine/system?id=wait)를 참조하십시오.

```php
Swoole\Process::wait(bool $blocking = true): array|false
```

* **매개변수**

  * **`bool $blocking`**
    * **기능**: 블로킹 대기 여부를 지정합니다.【기본값은 블로킹입니다.】
    * **기본값**: `true`
    * **다른 값**: `false`

* **반환 값**

  * 성공 시 하위 프로세스의 `PID`、종료 상태 코드 및 'KILL'된 시그널 정보를 포함한 배열이 반환됩니다.
  * 실패하면 `false`가 반환됩니다.

!> 각 하위 프로세스가 종료되면 부모 프로세스는 반드시 `wait()`를 실행하여 회수해야 합니다. 그렇지 않으면 하위 프로세스는 좀비 프로세스가 되어 운영 체제의 프로세스 리소스를 낭비하게 됩니다.  
부모 프로세스가 다른 작업을 수행해야 하고 `wait`를 블로킹할 수 없는 경우, 부모 프로세스는 종료된 프로세스에 대해 `SIGCHLD` 시그널을 등록해야 합니다.  
SIGCHILD 시그널이 발생하면 여러 개의 하위 프로세스가 동시에 종료될 수 있으며, `wait()`를 블로킹하지 않은 상태여야 하며, `false`가 반환될 때까지 `wait`를 반복 실행해야 합니다.

* **예시**

```php
Swoole\Process::signal(SIGCHLD, function ($sig) {
    // 블로킹 모드가 아닌 false여야 함
    while ($ret = Swoole\Process::wait(false)) {
        echo "PID={$ret['pid']}\n";
    }
});
```
### daemon()

현재 프로세스를 데몬 프로세스로 변환합니다.

```php
Swoole\Process::daemon(bool $nochdir = true, bool $noclose = true): bool
```

* **매개변수** 

  * **`bool $nochdir`**
    * **기능**：현재 디렉토리를 루트 디렉토리로 전환해야 하는지 여부【`true`이면 현재 디렉토리를 루트 디렉토리로 전환하지 않음】
    * **기본값**：`true`
    * **다른 값**：`false`

  * **`bool $noclose`**
    * **기능**：표준 입력 및 출력 파일 기술자를 닫아야 하는지 여부【`true`이면 표준 입력 및 출력 파일 기술자를 닫지 않음】
    * **기본값**：`true`
    * **다른 값**：`false`

!> 프로세스가 데몬 프로세스로 변환될 때 `PID`가 변경되며, 현재 `PID`를 얻으려면 `getmypid()`를 사용할 수 있습니다.
### alarm()

고정밀 타이머는 운영 체제의 `setitimer` 시스템 호출의 래핑으로, 마이크로 초 단위 타이머를 설정할 수 있습니다. 타이머는 시그널을 발생시키며 [Process::signal](/process/process?id=signal) 또는 `pcntl_signal`과 함께 사용해야 합니다.

!> `alarm`은 [Timer](/timer)와 동시에 사용할 수 없습니다.

```php
Swoole\Process->alarm(int $time, int $type = 0): bool
```

* **매개변수**

  * **`int $time`**
    * **기능**：타이머 간격 시간【음수이면 타이머 제거】
    * **값 단위**：마이크로 초
    * **기본값**：없음
    * **기타 값**：없음

  * **`int $type`**
    * **기능**：타이머 유형
    * **기본값**：`0`
    * **다른 값**：

타이머 유형 | 설명
---|---
0 | 실제 시간을 나타내며, `SIGALAM` 시그널을 발생시킵니다
1 | 사용자 공간 CPU 시간을 나타내며, `SIGVTALAM` 시그널을 발생시킵니다
2 | 사용자 및 커널 공간 시간을 나타내며, `SIGPROF` 시그널을 발생시킵니다

* **반환 값**

  * 설정 성공 시 `true` 반환
  * 실패 시 `false` 반환하며, 오류 코드는 `swoole_errno`을 통해 얻을 수 있습니다

* **사용 예시**

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

run(function () {
    Process::signal(SIGALRM, function () {
        static $i = 0;
        echo "#{$i}\talarm\n";
        $i++;
        if ($i > 20) {
            Process::alarm(-1);
            Process::kill(getmypid());
        }
    });

    //100밀리초
    Process::alarm(100 * 1000);

    while(true) {
        sleep(0.5);
    }
});
```
### setAffinity()

`CPU`의 친화성을 설정하여 프로세스를 특정 `CPU` 코어에 바인딩할 수 있습니다.

이 함수는 프로세스가 특정 `CPU` 코어에서만 실행되도록하고 일부 `CPU` 자원을 양보하여 더 중요한 프로그램을 실행할 수 있도록 합니다.

```php
Swoole\Process->setAffinity(array $cpus): bool
```

* **매개변수**

  * **`array $cpus`**
    * **기능**: `CPU` 코어에 바인딩합니다. 【예: `array(0,2,3)`은 `CPU0/CPU2/CPU3`에 바인딩하는 것을 의미합니다.】
    * **기본값**: 없음
    * **기타 값**: 없음

!> - `$cpus`의 요소 수는 `CPU` 코어 수를 초과할 수 없습니다.
- `CPU-ID`는 (`CPU` 코어 수 - `1`)을 초과할 수 없습니다.
- 이 함수는 운영 체제가 `CPU`를 바인딩할 수 있는 기능을 지원해야 합니다.
- [swoole_cpu_num()](/functions?id=swoole_cpu_num)를 사용하여 현재 서버의 `CPU` 코어 수를 얻을 수 있습니다.
### setPriority()

프로세스, 프로세스 그룹 및 사용자 프로세스의 우선 순위를 설정합니다.

!> Swoole version >= `v4.5.9`에서 사용 가능

```php
Swoole\Process->setPriority(int $which, int $priority): bool
```

* **매개변수** 

  * **`int $which`**
    * **기능**：우선순위를 변경할 유형을 결정합니다.
    * **기본값**：없음
    * **다른 값**：

| 상수         | 설명     |
| ------------ | -------- |
| PRIO_PROCESS | 프로세스     |
| PRIO_PGRP    | 프로세스 그룹   |
| PRIO_USER    | 사용자 프로세스 |

  * **`int $priority`**
    * **기능**：우선순위입니다. 값이 작을수록 우선순위가 높습니다.
    * **기본값**：없음
    * **다른 값**：`[-20, 20]`

* **반환 값**

  * `false`가 반환되면 [swoole_errno](/functions?id=swoole_errno) 및 [swoole_strerror](/functions?id=swoole_strerror)을 사용하여 오류 코드와 오류 정보를 얻을 수 있습니다.
### getPriority()

프로세스의 우선순위를 가져옵니다.

!> Swoole 버전 >= `v4.5.9`에서 사용 가능

```php
Swoole\Process->getPriority(int $which): int
```
