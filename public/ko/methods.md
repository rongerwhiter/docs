```python
# Methods
```

# 메서드
## __construct() 

[비동기 IO](/learn?id=동기io와비동기io) TCP Server 객체를 생성합니다.

```php
Swoole\Server::__construct(string $host = '0.0.0.0', int $port = 0, int $mode = SWOOLE_PROCESS, int $sockType = SWOOLE_SOCK_TCP): \Swoole\Server
```

  * **매개변수**

    * `string $host`

      * 기능: 리스닝할 IP 주소를 지정합니다.
      * 기본값: 없음.
      * 다른 값: 없음.

      !> IPv4의 경우, `127.0.0.1`은 로컬을 나타내며, `0.0.0.0`은 모든 주소를 의미합니다.
      IPv6의 경우, `::1`은 로컬을 나타내며, `::` (즉, `0:0:0:0:0:0:0:0`)는 모든 주소를 의미합니다.

    * `int $port`

      * 기능: 리스닝할 포트를 지정합니다. 예를 들어 `9501`.
      * 기본값: 없음.
      * 다른 값: 없음.

      !> `$sockType`의 값이 [UnixSocket Stream/Dgram](/learn?id=IPC란)인 경우, 이 매개변수는 무시됩니다.
      `1024` 미만의 포트를 리스닝하려면 `root` 권한이 필요합니다.
      이 포트가 이미 사용 중이면 `server->start` 시에 실패합니다.

    * `int $mode`

      * 기능: 실행 모드를 지정합니다.
      * 기본값: [SWOOLE_PROCESS](/learn?id=swoole_process) 다중 프로세스 모드 (기본값).
      * 다른 값: [SWOOLE_BASE](/learn?id=swoole_base) 베이스 모드.

      !> Swoole5 버전부터 실행 모드의 기본값은 `SWOOLE_BASE`입니다.

    * `int $sockType`

      * 기능: 이 그룹 Server의 유형을 지정합니다.
      * 기본값: 없음.
      * 다른 값:
        * `SWOOLE_TCP/SWOOLE_SOCK_TCP` tcp ipv4 소켓
        * `SWOOLE_TCP6/SWOOLE_SOCK_TCP6` tcp ipv6 소켓
        * `SWOOLE_UDP/SWOOLE_SOCK_UDP` udp ipv4 소켓
        * `SWOOLE_UDP6/SWOOLE_SOCK_UDP6` udp ipv6 소켓
        * [SWOOLE_UNIX_DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) 유닉스 소켓 dgram
        * [SWOOLE_UNIX_STREAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/stream_server.php) 유닉스 소켓 stream 

      !> `$sock_type` | `SWOOLE_SSL`를 사용하여 `SSL` 터널 암호화를 활성화할 수 있습니다. `SSL`을 활성화한 경우 반드시 설정해야 합니다. [ssl_key_file](/server/setting?id=ssl_cert_file)와 [ssl_cert_file](/server/setting?id=ssl_cert_file)

  * **예시**

```php
$server = new \Swoole\Server($host, $port = 0, $mode = SWOOLE_PROCESS, $sockType = SWOOLE_SOCK_TCP);

// UDP/TCP를 혼합해서 사용하고, 내부 및 외부 포트를 동시에 수신하는 여러 포트 수신에 대해서는 addlistener 섹션 참조.
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP); // TCP 추가
$server->addlistener("192.168.1.100", 9503, SWOOLE_SOCK_TCP); // 웹 소켓 추가
$server->addlistener("0.0.0.0", 9504, SWOOLE_SOCK_UDP); // UDP 추가
$server->addlistener("/var/run/myserv.sock", 0, SWOOLE_UNIX_STREAM); //UnixSocket Stream 추가
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP | SWOOLE_SSL); //TCP + SSL 추가

$port = $server->addListener("0.0.0.0", 0, SWOOLE_SOCK_TCP); // 시스템이 무작위로 포트를 할당하고 할당된 포트 값 반환
echo $port->port;
```
## set()

실행 시에 다양한 매개변수를 설정하는 데 사용됩니다. 서버가 시작된 후에 `$serv->setting`을 통해 `Server->set` 메서드로 설정된 매개변수 배열에 액세스할 수 있습니다.

```php
Swoole\Server->set(array $setting): void
```

!> `Server->set`은 `Server->start` 전에 호출해야하며, 각 설정 매개변수의 의미에 대해서는 [이 섹션](/server/setting)을 참조하십시오.

  * **예시**

```php
$server->set(array(
    'reactor_num'   => 2,     // 스레드 수
    'worker_num'    => 4,     // 프로세스 수
    'backlog'       => 128,   // Listen 대기열 길이 설정
    'max_request'   => 50,    // 각 프로세스당 최대 요청 수
    'dispatch_mode' => 1,     // 데이터 패킷 배포 전략
));
```
## on()

`Server`의 이벤트 콜백 함수를 등록합니다.

```php
Swoole\Server->on(string $event, callable $callback): bool
```

!> `on` 메서드를 여러 번 호출하면 이전 설정이 덮어씌워집니다.

!> PHP8.2에서는 `$event`가 `Swoole`에서 정의한 이벤트가 아닌 경우 PHP8.2가 경고를 발생시키며 직접적인 동적 속성 설정을 지원하지 않기 때문에 주의해야 합니다.

  * **매개변수**

    * `string $event`

      * 기능: 콜백 이벤트 이름
      * 기본값: 없음
      * 다른 값: 없음

      !> 대소문자가 구분되지 않으며, 사용 가능한 이벤트 콜백은 [이 섹션](/server/events)을 참조하십시오. 이벤트 이름 문자열에는 `on`을 추가하지 마십시오.

    * `callable $callback`

      * 기능: 콜백 함수
      * 기본값: 없음
      * 다른 값: 없음

      !> 함수 이름의 문자열, 클래스 정적 메서드, 객체 메서드 배열, 익명 함수가 될 수 있습니다. 자세한 내용은 [이 섹션](/learn?id=几种设置回调函数的方式)을 참조하십시오.
  
  * **반환값**

    * 반환값이 `true`이면 작업이 성공적으로 수행되었음을 나타내며, `false`이면 작업이 실패했음을 나타냅니다.

  * **예시**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('connect', function ($server, $fd){
    echo "Client:Connect.\n";
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->on('close', function ($server, $fd) {
    echo "Client: Close.\n";
});
$server->start();
```
## addListener()

대기 중인 포트를 추가합니다. 비즈니스 코드에서는 [Swoole\Server->getClientInfo](/server/methods?id=getclientinfo)을 호출하여 연결이 특정 포트에서 왔는지 확인할 수 있습니다.

```php
Swoole\Server->addListener(string $host, int $port, int $sockType): bool|Swoole\Server\Port
```

!> `1024` 미만의 포트를 청취하려면 `root` 권한이 필요합니다.  
주 서버가 `WebSocket` 또는 `HTTP` 프로토콜인 경우에는 새 `TCP` 포트가 기본적으로 주 서버의 프로토콜 설정을 상속받습니다. 새 프로토콜을 활성화하려면 별도로 `set` 메서드를 호출하여 새 프로토콜을 설정해야 합니다 [자세한 내용 확인하기](/server/port)  
`Swoole\Server\Port`에 대한 자세한 설명은 [여기](/server/server_port)를 클릭하여 볼 수 있습니다.

  * **Parameters**

    * `string $host`

      * Functionality: Same as `$host` in `__construct()`
      * Default value: Same as `$host` in `__construct()`
      * Other values: Same as `$host` in `__construct()`

    * `int $port`

      * Functionality: Same as `$port` in `__construct()`
      * Default value: Same as `$port` in `__construct()`
      * Other values: Same as `$port` in `__construct()`

    * `int $sockType`

      * Functionality: Same as `$sockType` in `__construct()`
      * Default value: Same as `$sockType` in `__construct()`
      * Other values: Same as `$sockType` in `__construct()`
  
  * **Return Value**

    * Returns `Swoole\Server\Port` if successful, returns `false` if failed.

!> - In `Unix Socket` mode, the `$host` parameter must be the accessible file path, and the `$port` parameter is ignored.  
- In `Unix Socket` mode, the client's `$fd` will no longer be a number but a string of a file path.  
- After listening on an `IPv6` port on a `Linux` system, you can also connect using an `IPv4` address.
```php
Swoole\Server->listen(string $host, int $port, int $type): bool|Swoole\Server\Port
```

## addProcess()

사용자 정의 작업 프로세스를 추가합니다. 이 함수는 보통 모니터링, 보고 또는 다른 특수 작업을 위한 특별한 작업 프로세스를 만드는 데 사용됩니다.

```php
Swoole\Server->addProcess(Swoole\Process $process): int
```

!> `start`를 실행할 필요가 없습니다. `Server`를 시작할 때 프로세스가 자동으로 생성되고 지정된 자식 프로세스 함수가 실행됩니다.

  * **매개변수**
  
    * [Swoole\Process](/process/process)

      * 기능: `Swoole\Process` 객체
      * 기본값: 없음
      * 다른 값: 없음

  * **반환 값**

    * 프로세스 ID 번호를 반환하여 작업이 성공했음을 나타냅니다. 그렇지 않으면 프로그램이 치명적인 오류를 인식합니다.

  * **주의**

    !> - 생성된 자식 프로세스는 `$server` 객체가 제공하는 여러 메소드를 호출할 수 있습니다. 예: `getClientList/getClientInfo/stats`         
    - `Worker/Task` 프로세스에서는 `$process`의 메소드를 호출하여 자식 프로세스와 통신할 수 있습니다.         
    - 사용자 정의 프로세스에서는 `$server->sendMessage`와 `Worker/Task` 프로세스와 통신할 수 있습니다.        
    - 사용자 프로세스 내에서는 `Server->task/taskwait` 인터페이스를 사용할 수 없습니다.       
    - 사용자 프로세스 내에서는 `Server->send/close`와 같은 인터페이스를 사용할 수 있습니다.        
    - 사용자 프로세스에서는 `while(true)`를 수행해야 하며(아래 예제 참조) 또는 [EventLoop](/learn?id=what-is-eventloop) 루프(예: 타이머 생성)를 만들어야 합니다. 그렇지 않으면 사용자 프로세스가 계속해서 종료 및 다시 시작됩니다.

  * **수명주기**

    ?> - 사용자 프로세스의 수명은 `Master` 및 [Manager](/learn?id=manager-process)와 동일하며, [reload](/server/methods?id=reload)에 영향을 받지 않습니다.      
    - 사용자 프로세스는 `reload` 명령으로 제어되지 않으며, `reload` 시에 사용자 프로세스로 어떠한 정보도 전송되지 않습니다.       
    - 서버를 `shutdown`하여 종료할 때, 사용자 프로세스에 `SIGTERM` 신호를 보내어 사용자 프로세스를 종료합니다.      
    - 사용자 정의 프로세스는 `Manager` 프로세스에 의해 관리되며, 치명적인 오류가 발생하면 `Manager` 프로세스가 새로 생성됩니다.      
    - 사용자 정의 프로세스는 `onWorkerStop` 등의 이벤트를 트리거하지 않습니다. 

  * **예시**

    ```php
    $server = new Swoole\Server('127.0.0.1', 9501);
    
    /**
     * 사용자 프로세스는 broadcast 기능을 구현한다. unixSocket의 메시지를 계속 수신하고 서버의 모든 연결에게 전송한다.
     */
    $process = new Swoole\Process(function ($process) use ($server) {
        $socket = $process->exportSocket();
        while (true) {
            $msg = $socket->recv();
            foreach ($server->connections as $conn) {
                $server->send($conn, $msg);
            }
        }
    }, false, 2, 1);
    
    $server->addProcess($process);
    
    $server->on('receive', function ($serv, $fd, $reactor_id, $data) use ($process) {
        // Receives the message and sends it to all connections
        $socket = $process->exportSocket();
        $socket->send($data);
    });
    
    $server->start();
    ```

    [프로세스 간 통신 섹션](/process/process?id=exportsocket)을 참조하십시오.
## start()

서버를 시작하여 모든 `TCP/UDP` 포트를 수신 대기합니다.

```php
Swoole\Server->start(): bool
```

!> 참고: 아래 예시는 [SWOOLE_PROCESS](/learn?id=swoole_process) 모드를 기준으로 합니다.

  * **팁**

    - 성공적으로 시작하면 `worker_num+2`개의 프로세스가 생성됩니다. `Master` 프로세스 + `Manager` 프로세스 + `serv->worker_num`개의 `Worker` 프로세스입니다.
    - 시작에 실패하면 즉시 `false`를 반환합니다.
    - 성공적으로 시작하면 이벤트 루프에 진입하여 클라이언트의 연결 요청을 대기합니다. `start` 메소드 이후의 코드는 실행되지 않습니다.
    - 서버가 닫힌 후에는 `start` 함수가 `true`를 반환하고 계속해서 실행됩니다.
    - `task_worker_num`을 설정하면 해당 수만큼의 [Task 프로세스](/learn?id=taskworker프로세스)가 추가됩니다.
    - 메소드 목록 중 `start` 이전의 메소드는 `start` 호출 전에만 사용할 수 있으며, `start` 이후의 메소드는 [onWorkerStart](/server/events?id=onworkerstart) 등 이벤트 콜백 함수 내에서만 사용할 수 있습니다.

  * **확장**

    * Master 주 프로세스

      * 주 프로세스에는 여러 개의 [Reactor](/learn?id=reactor스레드) 스레드가 있으며, `epoll/kqueue/select`를 기반으로 네트워크 이벤트를 폴링합니다. 데이터를 받으면 `Worker` 프로세스로 전달하여 처리합니다.
    
    * Manager 프로세스

      * 모든 `Worker` 프로세스를 관리하며, `Worker` 프로세스의 수명이 종료되거나 예외가 발생하면 자동으로 회수하고 새로운 `Worker` 프로세스를 생성합니다.
    
    * Worker 프로세스

      * 받은 데이터를 처리하며, 프로토콜 해석 및 요청 응답을 포함합니다. `worker_num`을 설정하지 않으면, 하위 레벨에서는 `CPU` 수와 동일한 수의 `Worker` 프로세스를 시작합니다.
      * 시작이 실패하면 확장 내에서 치명적인 오류가 발생하므로, `php error_log`의 관련 정보를 확인해야 합니다. `errno={number}`는 표준 `Linux Errno`이며, 관련 문서를 참조할 수 있습니다.
      * `log_file` 설정을 활성화한 경우, 정보는 지정된 `Log` 파일에 기록됩니다.

  * **반환값**

    * `true`를 반환하면 작업이 성공적으로 수행되었음을 의미하며, `false`를 반환하면 작업이 실패했음을 의미합니다.

  * **시작 실패의 일반적인 원인**

    * `bind` 포트에 실패했으며, 다른 프로세스가 해당 포트를 이미 점유하고 있습니다.
    * 필수 콜백 함수가 설정되지 않아 시작에 실패했습니다.
    * `PHP` 코드에 치명적인 오류가 있는 경우, `php_errors.log`를 확인해야 합니다.
    * `ulimit -c unlimited`을 실행하여 `core dump`를 열고, 세그멘테이션 오류가 있는지 확인해야 합니다.
    * `daemonize`를 중지하고 `log`를 중지하여 오류 정보를 화면에 인쇄할 수 있도록 설정합니다.
## reload()

모든 Worker/Task 프로세스를 안전하게 다시 시작합니다.

```php
Swoole\Server->reload(bool $only_reload_taskworker = false): bool
```

!> 예시: 바쁜 백엔드 서버는 언제든지 요청을 처리하고 있습니다. 만약 관리자가 `kill` 프로세스 방식으로 서버 프로그램을 중지/다시 시작한다면, 정확히 코드가 실행 중일 때 중지될 수 있습니다.  
이러한 상황에서 데이터의 불일치가 발생할 수 있습니다. 예를 들어, 결제 로직 다음에 배송 로직이 있는 거래 시스템에서, 결제 로직 이후에 프로세스가 중지된다고 가정해보겠습니다. 이는 사용자가 통화를 결제했지만 물건을 전달받지 못하는 심각한 결과를 낳을 수 있습니다.  
`Swoole`은 부드러운 중지/다시 시작 메커니즘을 제공하여, 관리자가 `Server`에 특정 신호를 보내기만 하면, `Server`의 `Worker` 프로세스가 안전하게 종료됩니다. 참고: [서비스를 올바르게 다시 시작하는 방법](/question/use?id=swoole如何正确的重启服务)。

  * **매개변수**
  
    * `bool $only_reload_taskworker`

      * 기능: [Task 프로세스](/learn?id=taskworker进程)만 다시 시작할지 여부
      * 기본값: false
      * 다른 값: true

!> -`reload`에는 보호 메커니즘이 있어 한 번에 `reload`가 진행 중일 때 새로운 재시작 신호를 수신하면 폐기됩니다.
-만약 `user/group`가 설정되어 있을 경우, `Worker` 프로세스는 `master` 프로세스에 정보를 보낼 권한이 없을 수 있습니다. 이 경우에는 `root` 계정을 사용하여 `shell`에서 `kill` 명령을 실행해야 합니다.
-`reload` 명령은 [addProcess](/server/methods?id=addProcess)로 추가된 사용자 프로세스에는 적용되지 않습니다.

  * **반환 값**

    * 성공하면 `true`를 반환하고, 실패하면 `false`를 반환합니다.
       
  * **확장**
  
    * **신호 보내기**
    
        * `SIGTERM`: 이 신호를 주요 프로세스/관리 프로세스에 보내면 서버가 안전하게 종료됩니다.
        * PHP 코드에서 이 작업을 수행하려면 `$serv->shutdown()`을 호출하십시오.
        * `SIGUSR1`: 주요 프로세스/관리 프로세스에 `SIGUSR1` 신호를 보내면 모든 `Worker` 프로세스와 `TaskWorker` 프로세스를 부드럽게 `restart`합니다.
        * `SIGUSR2`: 주요 프로세스/관리 프로세스에 `SIGUSR2` 신호를 보내면 모든 `Task` 프로세스를 부드럽게 다시 시작합니다.
        * PHP 코드에서 이 작업을 수행하려면 `$serv->reload()`를 호출하십시오.
        
    ```shell
    # 모든 worker 프로세스 다시 시작
    kill -USR1 main_process_PID
    
    # task 프로세스만 다시 시작
    kill -USR2 main_process_PID
    ```
      
      > [참고: Linux 신호 목록](/other/signal)

    * **Process 모드**
    
        `Process`로 시작된 프로세스에서, 클라이언트로부터의 `TCP` 연결은 `Master` 프로세스 내에서 유지되며, `Worker` 프로세스의 다시 시작 및 비정상 종료는 연결 자체에 영향을 주지 않습니다.

    * **Base 모드**
    
        `Base` 모드에서는 클라이언트 연결이 직접 `Worker` 프로세스에서 유지되므로, `reload`시 모든 연결이 끊어집니다.

    !> `Base` 모드는 [Task 프로세스](/learn?id=taskworker进程)의 `reload`를 지원하지 않습니다.
    
    * **Reload 유효 범위**

      `Reload` 작업은 `Worker` 프로세스가 시작된 후 로드된 PHP 파일들만 다시로드할 수 있으며, `get_included_files` 함수를 사용하여 `WorkerStart` 이전에 로드된 PHP 파일 목록을 나열할 수 있습니다. 이 목록에 있는 PHP 파일은 `reload` 작업을 수행해도 다시로드할 수 없습니다. 서버를 다시 시작해야 적용됩니다.

    ```php
    $serv->on('WorkerStart', function(Swoole\Server $server, int $workerId) {
        var_dump(get_included_files()); // 이 배열에 있는 파일들은 프로세스 시작 전에 이미 로드된 것이므로 다시로드할 수 없습니다
    });
    ```

    * **APC/OPcache**
    
        만약 `PHP`에서 `APC/OPcache`를 활성화했다면, `reload` 다시로드 시 영향을 받을 수 있습니다. 이 문제에 대한 해결 방법은 2가지가 있습니다.
        
        * `APC/OPcache`의 `stat` 검사를 활성화하면 파일이 업데이트되면 `APC/OPcache`가 자동으로 `OPCode`를 업데이트합니다.
        * `onWorkerStart`에서 파일을 로드하기 전에 `apc_clear_cache`나 `opcache_reset`을 실행하여 `OPCode` 캐시를 새로 고칩니다.

  * **주의 사항**

    !> -부드러운 다시 시작은 `Worker` 프로세스에서 `include/require`한 [onWorkerStart](/server/events?id=onworkerstart) 또는 [onReceive](/server/events?id=onreceive)와 같은 PHP 파일에만 적용됩니다.
    -`Server`가 시작하기 전에 `include/require`한 PHP 파일은 부드러운 다시 시작으로 다시로드할 수 없습니다.
    -`Server` 설정인 `$serv->set()`에 전달된 매개변수 설정은 서버를 닫거나 다시 시작해야만 다시로드됩니다.
    -`Server`는 내부 네트워크 포트를 수신하여 원격 제어 명령을 수신한 후 모든 `Worker` 프로세스를 다시 시작할 수 있습니다.
## stop()

현재 `Worker` 프로세스를 중지하고 즉시 `onWorkerStop` 콜백 함수를 트리거합니다.

```php
Swoole\Server->stop(int $workerId = -1, bool $waitEvent = false): bool
```

  * **매개변수**

    * `int $workerId`

      * 기능: `worker id`를 지정합니다.
      * 기본값: -1, 현재 프로세스를 의미합니다.
      * 다른 값: 없음

    * `bool $waitEvent`

      * 기능: 종료 전략을 제어합니다. `false`는 즉시 종료를 의미하고, `true`는 이벤트 루프가 비어있을 때에만 종료를 의미합니다.
      * 기본값: false
      * 다른 값: true

  * **반환값**

    * 성공적으로 작업을 한 경우 `true`를 반환하고, 실패한 경우 `false`를 반환합니다.

  * **팁**

    !> -[비동기 I/O](/learn?id=동기io비동기io) 서버는 `stop`을 호출하여 프로세스를 종료할 때까지 대기 중인 이벤트가 여전히 있을 수 있습니다. 예를 들어 `Swoole\MySQL->query`를 사용하여 `SQL` 쿼리를 보냈지만 아직 `MySQL` 서버로부터 결과를 기다리고 있는 경우입니다. 이때 프로세스를 강제로 종료하면 `SQL` 실행 결과가 손실될 수 있습니다.  
    -`$waitEvent = true`로 설정하면 하부에서 [비동기 안전 재시작](/question/use?id=swoole如何正确的重启服务) 전략을 사용합니다. 먼저 `Manager` 프로세스에게 알리고 새 `Worker`를 시작하여 새 요청을 처리합니다. 현재 오래된 `Worker`는 이벤트를 대기하고 있습니다. 이벤트 루프가 비어 있거나 `max_wait_time`을 초과하면 프로세스를 종료하여 비동기 이벤트의 안전을 최대한 보장합니다.
## shutdown()

서비스를 종료합니다.

```php
Swoole\Server->shutdown(): bool
```

  * **반환 값**

    * `true`를 반환하면 작업이 성공적이며, `false`를 반환하면 작업이 실패한 것입니다.

  * **팁**

    * 이 함수는 `Worker` 프로세스 내에서 사용할 수 있습니다.
    * 마스터 프로세스에 `SIGTERM`을 보내어도 서비스를 종료할 수 있습니다.

```shell
kill -15 마스터프로세스PID
```
## tick()

`tikc` 타이머를 추가하고 사용자 정의 콜백 함수를 설정할 수 있습니다. 이 함수는 [Swoole\Timer::tick](/timer?id=tick)의 별칭입니다.

```php
Swoole\Server->tick(int $millisecond, callable $callback): void
```

  * **매개변수**

    * `int $millisecond`

      * 기능: 간격 시간【밀리초】
      * 기본값: 없음
      * 다른 값: 없음

    * `callable $callback`

      * 기능: 콜백 함수
      * 기본값: 없음
      * 다른 값: 없음

  * **주의**

    !> -`Worker` 프로세스가 종료되면 모든 타이머가 자동으로 소멸됩니다.
    -`tick/after` 타이머는 `Server->start` 이전에 사용할 수 없습니다.
    -`Swoole5` 이후에는이 별칭 사용 방법이 삭제되었으므로 직접 `Swoole\Timer::tick()`을 사용하십시오.

  * **예제**

    * [onReceive](/server/events?id=onreceive)에서 사용

    ```php
    function onReceive(Swoole\Server $server, int $fd, int $reactorId, mixed $data)
    {
        $server->tick(1000, function () use ($server, $fd) {
            $server->send($fd, "hello world");
        });
    }
    ```

    * [onWorkerStart](/server/events?id=onworkerstart)에서 사용

    ```php
    function onWorkerStart(Swoole\Server $server, int $workerId)
    {
        if (!$server->taskworker) {
            $server->tick(1000, function ($id) {
              var_dump($id);
            });
        } else {
            //task
            $server->tick(1000);
        }
    }
    ```
## after()

한 번만 실행되고 완료되면 파괴되는 일회성 타이머를 추가합니다. 이 함수는 [Swoole\Timer::after](/timer?id=after)의 별칭입니다.

```php
Swoole\Server->after(int $millisecond, callable $callback)
```

  * **매개변수**

    * `int $millisecond`

      * 기능: 실행 시간 [밀리초]
      * 기본값: 없음
      * 다른 값: 없음
      * 버전 영향: `Swoole v4.2.10` 미만 버전에서는 최대값이 `86400000`을 초과할 수 없음

    * `callable $callback`

      * 기능: 콜백 함수, 호출 가능해야 함, `callback` 함수는 어떤 매개변수도 받지 않음
      * 기본값: 없음
      * 다른 값: 없음

  * **주의사항**
  
    !> - 타이머의 수명은 프로세스 수준이며 `reload` 또는 `kill`을 사용하여 프로세스를 재시작 또는 종료할 때, 타이머가 모두 파괴됩니다  
    - 어떤 타이머가 중요한 로직 및 데이터를 가지고 있다면 `onWorkerStop` 콜백 함수에서 구현하거나 [서버를 올바르게 다시 시작하는 방법](/question/use?id=swoole如何正确的重启服务)을 참고하세요  
    - `Swoole5` 이후, 이 별칭 사용 방법이 삭제되었으므로, 직접 `Swoole\Timer::after()`를 사용하세요
## defer()

[Swoole\Event::defer](/event?id=defer)의 별칭으로 함수를 나중에 실행합니다.

```php
Swoole\Server->defer(Callable $callback): void
```

  * **파라미터**

    * `Callable $callback`

      * 기능: 콜백 함수【필수】, 실행 가능한 함수 변수, 문자열, 배열, 익명 함수일 수 있음
      * 기본 값: 없음
      * 기타 값: 없음

  * **주의**

    !> - 이 함수는 [EventLoop](/learn?id=이벤트루프) 루프가 완료된 후에 실행됩니다. 이 함수의 목적은 일부 PHP 코드를 나중에 실행하도록하여 프로그램이 다른 'IO' 이벤트를 우선 처리할 수 있게 하는 것입니다. 예를 들어 CPU 집약적인 계산이 필요하지만 급하게 처리할 필요가 없는 콜백 함수가 있을 경우, 프로세스가 다른 이벤트를 처리한 후 CPU 집약적인 계산을 할 수 있습니다.  
    - `defer` 함수가 즉시 실행되는 것을 보장하지 않습니다. 시스템의 핵심 로직이라면 빠르게 실행해야하는 경우 `after` 타이머를 사용해 구현해야 합니다.  
    - `onWorkerStart` 콜백 내에서 `defer`를 실행할 때는 이벤트가 발생해야 콜백이 호출됩니다.  
    - `Swoole5` 이후, 이 별칭 사용 방법은 삭제되었으므로 바로 `Swoole\Event::defer()`를 사용하세요.

  * **예시**

```php
function query($server, $db) {
    $server->defer(function() use ($db) {
        $db->close();
    });
}
```
## clearTimer()

`tck/after` 타이머를 지우는 함수로, 이 함수는 [Swoole\Timer::clear](/timer?id=clear)의 별칭입니다.

```php
Swoole\Server->clearTimer(int $timerId): bool
```

  * **매개변수**

    * `int $timerId`

      * 기능: 지정된 타이머 id
      * 기본값: 없음
      * 다른 값: 없음

  * **반환 값**

    * `true`를 반환하면 작업이 성공적으로 완료됐음을 의미하고, `false`를 반환하면 작업이 실패했음을 의미합니다.

  * **주의 사항**

    !> -`clearTimer`는 현재 프로세스의 타이머만 지울 수 있습니다.    
    -`Swoole5` 이후, 이 별칭 사용 방법이 제거되었으니 `Swoole\Timer::clear()`를 직접 사용해야 합니다.

  * **예시**

```php
$timerId = $server->tick(1000, function ($timerId) use ($server) {
    $server->clearTimer($timerId);//$id는 타이머의 id입니다
});
```
## close()

클라이언트 연결을 닫습니다.

```php
Swoole\Server->close(int $fd, bool $reset = false): bool
```

* **매개변수**

  * `int $fd`

    * 기능: 닫을 `fd` (파일 기술자) 지정
    * 기본값: 없음
    * 기타 값: 없음

  * `bool $reset`

    * 기능: 이 값을 `true`로 설정하면 연결을 강제로 종료하고 보내지 않은 데이터가 포함된 송신 대기열이 폐기됩니다.
    * 기본값: false
    * 기타 값: true

* **반환값**

  * `true`를 반환하면 작업 성공, `false`를 반환하면 작업 실패

* **주의사항**

  !> - `Server`가 연결을`close`한 경우에도 [onClose](/server/events?id=onclose) 이벤트가 발생합니다.
  - `close` 후에 정리 로직을 작성하지 마십시오. 이것은 [onClose](/server/events?id=onclose) 콜백에 처리해야 합니다.
  - `HTTP\Server`의 `fd`는 상위 콜백 방법의 `response`에서 가져옵니다.

* **예시**

```php
$server->on('request', function ($request, $response) use ($server) {
    $server->close($response->fd);
});
```
## send()

클라이언트에 데이터를 보냅니다.

```php
Swoole\Server->send(int|string $fd, string $data, int $serverSocket = -1): bool
```

  * **매개변수**

    * `int|string $fd`

      * 기능: 클라이언트의 파일 디스크립터 또는 유닉스 소켓 경로를 지정합니다.
      * 기본값: 없음
      * 다른 값: 없음

    * `string $data`

      * 기능: 보낼 데이터입니다. `TCP` 프로토콜의 경우 최대 `2M`를 초과하면 안 되며, [buffer_output_size](/server/setting?id=buffer_output_size)를 수정하여 보낼 수 있는 최대 패킷 길이를 변경할 수 있습니다.
      * 기본값: 없음
      * 다른 값: 없음

    * `int $serverSocket`

      * 기능: [UnixSocket DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php)의 대상에 데이터를 보낼 때 이 매개변수가 필요하며, TCP 클라이언트는 작성할 필요가 없습니다.
      * 기본값: -1, 현재 듣는 udp 포트를 나타냄
      * 다른 값: 없음

  * **반환 값**

    * `true`를 반환하면 작업이 성공적으로 수행되었음을 나타내며, `false`를 반환하면 작업이 실패했음을 나타냅니다.

  * **팁**

    !> 전송 프로세스는 비동기적이며, 하부에서는 쓰기 가능한 상태를 자동으로 듣고 데이터를 클라이언트에 점진적으로 보냅니다. 즉, `send`가 반환된 후에 대상이 데이터를받지는 않습니다.

    * 보안
      * `send` 작업은 원자적입니다. 여러 프로세스가 동시에 동일한 `TCP` 연결로 데이터를 보내도 데이터가 뒤섞이지 않습니다.

    * 길이 제한
      * `2M`를 초과하는 데이터를 전송해야 하는 경우, 데이터를 임시 파일에 쓴 다음 `sendfile` 인터페이스를 통해 전송할 수 있습니다.
      * [buffer_output_size](/server/setting?id=buffer_output_size) 매개변수를 설정하여 전송 길이 제한을 수정할 수 있습니다.
      * `8K`를 초과하는 데이터를 전송하는 경우, 하부에서는 `Worker` 프로세스의 공유 메모리를 활성화하고 한 번에 한 번의 `Mutex->lock` 작업이 필요합니다.

    * 버퍼 영역
      * `Worker` 프로세스의 [유닉스 소켓](/learn?id=IPC) 버퍼가 가득 찼을 때, `8K` 데이터를 전송하면 임시 파일이 사용됩니다.
      * 동일한 클라이언트로 연속해서 대량의 데이터를 보낼 때, 클라이언트가 수신하는 데 시간이 오래 걸리면 `Socket` 메모리 버퍼가 꽉 찰 수 있어 Swoole 하부에서 즉시 `false`를 반환하고, 이때는 데이터를 디스크에 저장하고, 클라이언트가 이미 받은 데이터를 다시 보내기를 기다린 후 전송을 계속할 수 있습니다.

    * [코루틴 스케줄링](/coroutine?id=코루틴_스케줄링)
      * `send_yield`를 활성화한 경우 캐쉬 레벨이 가득 찼을 때 `send`는 자동으로 일시 중단되며, 대상이 데이터를 일부 소비한 후에 코루틴을 재개하여 데이터를 계속 보냅니다.

    * [유닉스 소켓](/learn?id=IPC)
      * `UnixSocket DGRAM` 포트를 듣고 있을 때, `send`를 사용하여 대상에 데이터를 보낼 수 있습니다.

      ```php
      $server->on("packet", function (Swoole\Server $server, $data, $addr){
          $server->send($addr['address'], 'SUCCESS', $addr['server_socket']);
      });
      ```  
## sendfile()

`TCP` 클라이언트 연결로 파일을 보냅니다.

```php
Swoole\Server->sendfile(int $fd, string $filename, int $offset = 0, int $length = 0): bool
```

  * **매개변수**

    * `int $fd`

      * 기능: 클라이언트의 파일 설명자를 지정합니다
      * 기본값: 없음
      * 다른 값: 없음

    * `string $filename`

      * 기능: 전송할 파일 경로이며 파일이 없는 경우 `false`를 반환합니다
      * 기본값: 없음
      * 다른 값: 없음

    * `int $offset`

      * 기능: 파일 오프셋을 지정하여 파일의 특정 위치부터 데이터를 보낼 수 있습니다
      * 기본값: 0 【기본값은`0`으로 파일 헤더부터 전송을 의미합니다】
      * 다른 값: 없음

    * `int $length`

      * 기능: 전송할 길이를 지정합니다
      * 기본값: 파일 크기
      * 다른 값: 없음

  * **반환 값**

    * `true`를 반환하면 작업 성공을, `false`를 반환하면 작업 실패를 나타냅니다

  * **주의 사항**

  !> 이 함수(`Server->sendfile`) 및 `Server->send`는 모두 클라이언트에 데이터를 보내지만, `sendfile`은 특정 파일에서 데이터를 가져옵니다.
## sendto()

클라이언트 `IP:PORT`로 `UDP` 데이터 패킷을 보냅니다.

```php
Swoole\Server->sendto(string $ip, int $port, string $data, int $serverSocket = -1): bool
```

  * **Parameters**

    * `string $ip`

      * 기능: 클라이언트 `ip` 지정
      * 기본값: 없음
      * 다른 값: 없음

      ?> `$ip`는 `IPv4` 또는 `IPv6` 문자열이어야 합니다. 잘못된 `IP`일 경우 오류가 발생합니다.

    * `int $port`

      * 기능: 클라이언트 `port` 지정
      * 기본값: 없음
      * 다른 값: 없음

      ?> `$port`는 `1-65535` 범위의 네트워크 포트 번호여야 합니다. 잘못된 포트로 전송하면 실패합니다.

    * `string $data`

      * 기능: 전송할 데이터 내용, 텍스트 또는 이진 콘텐츠가 될 수 있음
      * 기본값: 없음
      * 다른 값: 없음

    * `int $serverSocket`

      * 기능: 데이터 패킷을 보내는 데 사용할 포트를 지정하는 서버 소켓 디스크립터`server_socket`
      * 기본값: -1, 현재 수신 대기 중인 udp 포트를 의미
      * 다른 값: 없음

  * **Return Value**

    * `true` 반환 시 작업 성공, `false` 반환 시 작업 실패

      ?> 서버는 여러 `UDP` 포트를 동시에 수신할 수 있습니다. [다중 포트 수신](/server/port)을 참조하여 데이터 패킷을 전송할 포트를 지정할 수 있습니다.

  * **Note**

  !> `UDP` 포트를 수신해야만 `IPv4` 주소로 데이터를 보낼 수 있습니다.  
  `UDP6` 포트를 수신해야만 `IPv6` 주소로 데이터를 보낼 수 있습니다.

  * **Example**

```php
// IP 주소가 220.181.57.216인 호스트의 9502 포트로 "hello world" 문자열을 보냅니다.
$server->sendto('220.181.57.216', 9502, "hello world");
// IPv6 서버로 UDP 데이터 패킷을 보냅니다.
$server->sendto('2600:3c00::f03c:91ff:fe73:e98f', 9501, "hello world");
```
## sendwait()

데이터를 클라이언트에 동기적으로 전송합니다.

```php
Swoole\Server->sendwait(int $fd, string $data): bool
```

  * **매개변수**

    * `int $fd`

      * 기능: 클라이언트의 파일 디스크립터 지정
      * 기본값: 없음
      * 다른 값: 없음

    * `string $data`

      * 기능: 전송할 데이터
      * 기본값: 없음
      * 다른 값: 없음

  * **반환 값**

    * `true`를 반환하면 작업이 성공적으로 완료됨을 나타내고, `false`를 반환하면 작업이 실패함을 나타냄

  * **팁**

    * 특정 상황에서 `Server`는 클라이언트로 데이터를 연속적으로 전송해야 할 수도 있습니다. 그러나 `Server->send` 데이터 전송 인터페이스는 완전히 비동기적이므로 대량의 데이터 전송은 메모리 전송 대기열이 가득 차는 문제를 발생시킵니다.

    * `Server->sendwait`를 사용하면 이 문제를 해결할 수 있습니다. `Server->sendwait`는 연결이 쓰기 가능해질 때까지 대기합니다. 데이터가 완전히 전송될 때까지 반환되지 않습니다.

  * **주의사항**

  !> `sendwait`는 현재 [SWOOLE_BASE](/learn?id=swoole_base) 모드에서만 사용 가능합니다.  
  `sendwait`는 내부 또는 내부 네트워크 통신에만 사용되어야 합니다. 외부 연결에는 `sendwait`를 사용하지 말아야 하며, `enable_coroutine` => true(기본 활성화)일 때 이 함수를 사용하지 마십시오. 다른 코루틴을 막을 수 있습니다. 동기적 블로킹 서버에서만 사용할 수 있습니다.
## sendMessage()

임의의 `Worker` 프로세스나 [Task 프로세스](/learn?id=taskworker프로세스)로 메시지를 보냅니다. 메인 프로세스나 관리 프로세스 이외에서 호출할 수 있습니다. 메시지를 받은 프로세스는 `onPipeMessage` 이벤트를 트리거합니다.

```php
Swoole\Server->sendMessage(mixed $message, int $workerId): bool
```

  * **매개변수**

    * `mixed $message`

      * 기능: 전송할 메시지 데이터 내용, 길이 제한은 없지만 `8K`를 초과하면 메모리 임시 파일이 생성됨
      * 기본값: 없음
      * 다른 값: 없음

    * `int $workerId`

      * 기능: 대상 프로세스의 `ID`, 범위는 [$worker_id](/server/properties?id=worker_id) 참조
      * 기본값: 없음
      * 다른 값: 없음

  * **팁**

    * `Worker` 프로세스 내에서 `sendMessage`를 호출하는 것은 [비동기 IO](/learn?id=동기io비동기io)이며, 메시지는 먼저 버퍼에 저장되어 쓸 준비가 되면 이를 [unixSocket](/learn?id=ipc란)으로 전송합니다.
    * [Task 프로세스](/learn?id=taskworker프로세스) 내에서 `sendMessage`를 호출하는 것은 기본적으로 [동기 IO](/learn?id=동기io비동기io)이지만 일부 상황에서 자동으로 비동기 IO로 전환됩니다. 자세한 내용은 [동기 IO를 비동기 IO로 전환](/learn?id=동기io를-비동기-io로-전환)을 참조하세요.
    * [사용자 프로세스](/server/methods?id=addprocess) 내에서 `sendMessage`를 호출하는 것은 Task와 동일하게 기본적으로 동기 블록됩니다. 자세한 내용은 [동기 IO를 비동기 IO로 전환](/learn?id=동기io를-비동기-io로-전환)을 참조하세요.

  * **주의**

  !> - 만약 `sendMessage()`가 [비동기 IO](/learn?id=동기io를-비동기-io로-전환)이고 상대방 프로세스가 어떤 이유로든 데이터를 수신하지 않을 경우 `sendMessage()`를 계속 호출해서는 안 됩니다. 이렇게 하면 많은 메모리 자원을 사용하게 됩니다. 대신 상대방이 응답하지 않으면 호출을 일시 중지하는 응답 메커니즘을 추가할 수 있습니다.
-`MacOS/FreeBSD`에서 `2K`를 초과하면 임시 파일을 사용하여 저장됩니다.
-[sendMessage](/server/methods?id=sendMessage)를 사용하려면 `onPipeMessage` 이벤트 콜백 함수를 등록해야 합니다.
-[task_ipc_mode](/server/setting?id=task_ipc_mode)를 `3`으로 설정하면 특정 Task 프로세스로 메시지를 보낼 수 없습니다.

  * **예시**

```php
$server = new Swoole\Server('0.0.0.0', 9501);

$server->set(array(
    'worker_num'      => 2,
    'task_worker_num' => 2,
));
$server->on('pipeMessage', function ($server, $src_worker_id, $data) {
    echo "#{$server->worker_id} message from #$src_worker_id: $data\n";
});
$server->on('task', function ($server, $task_id, $src_worker_id, $data) {
    var_dump($task_id, $src_worker_id, $data);
});
$server->on('finish', function ($server, $task_id, $data) {

});
$server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
    if (trim($data) == 'task') {
        $server->task("async task coming");
    } else {
        $worker_id = 1 - $server->worker_id;
        $server->sendMessage("hello task process", $worker_id);
    }
});

$server->start();
```
## exist()

`fd`에 해당하는 연결의 존재 여부를 확인합니다.

```php
Swoole\Server->exist(int $fd): bool
```

  * **매개변수**

    * `int $fd`

      * 기능: 파일 디스크립터
      * 기본값: 없음
      * 다른 값: 없음

  * **반환 값**

    * `true`가 반환되면 존재, `false`가 반환되면 존재하지 않음을 나타냅니다.

  * **팁**
  
    * 이 인터페이스는 공유 메모리를 기반으로 계산되며, 어떤 `IO` 작업도 수행되지 않습니다.
## pause()

데이터 수신을 중단합니다.

```php
Swoole\Server->pause(int $fd): bool
```

  * **매개변수**

    * `int $fd`

      * 기능: 파일 기술자를 지정합니다.
      * 기본값: 없음
      * 다른 값: 없음

  * **반환 값**

    * `true`를 반환하면 작업이 성공적으로 완료되었음을 나타냅니다. `false`를 반환하면 작업이 실패했음을 나타냅니다.

  * **팁**

    * 이 함수를 호출하면 연결이 [이벤트 루프](/learn?id=什么是eventloop)에서 제거되어 클라이언트 데이터를 더 이상 수신하지 않습니다.
    * 이 함수는 전송 대기열을 처리하는 데 영향을 주지 않습니다.
    * `SWOOLE_PROCESS` 모드에서만 사용할 수 있습니다. `pause`를 호출한 후에는 일부 데이터가 `Worker` 프로세스에 도달했을 수 있으므로 여전히 [onReceive](/server/events?id=onreceive) 이벤트가 발생할 수 있습니다.
## resume()

데이터 수신을 재개합니다. `pause` 메서드와 짝을 이뤄 사용합니다.

```php
Swoole\Server->resume(int $fd): bool
```

  * **매개변수**

    * `int $fd`

      * 기능: 파일 디스크립터를 지정합니다.
      * 기본값: 없음
      * 다른 값: 없음

  * **반환 값**

    * `true`를 반환하면 작업이 성공하였음을 나타내며, `false`를 반환하면 작업이 실패하였음을 나타냅니다.

  * **팁**

    * 이 함수를 호출하면 연결이 [EventLoop](/learn?id=이벤트-루프란 무엇인가)에 다시 추가되어 클라이언트 데이터를 계속 수신합니다.
## getCallback()

서버의 지정된 이름의 콜백 함수를 가져옵니다.

```php
Swoole\Server->getCallback(string $event_name): \Closure|string|null|array
```

* **매개변수**

  * `string $event_name`

    * 기능: 이벤트 이름을 나타냅니다. `on`을 추가할 필요가 없으며 대소문자를 구분하지 않습니다.
    * 기본값: 없음
    * 다른 값: [이벤트](/server/events) 참조

* **반환값**

  * 해당 콜백 함수가 있는 경우, [콜백 함수 설정 방식](/learn?id=四种设置回调函数的方式)에 따라 `Closure` / `string` / `array`를 반환합니다.
  * 해당 콜백 함수가 없는 경우, `null`을 반환합니다.
## getClientInfo()

연결 정보를 가져오는 메소드로, 별칭은 `Swoole\Server->connection_info()`입니다.

```php
Swoole\Server->getClientInfo(int $fd, int $reactorId = -1, bool $ignoreError = false): false|array
```

  * **Parameters**

    * `int $fd`

      * 기능: 지정된 파일 디스크립터
      * 기본값: 없음
      * 다른 값: 없음

    * `int $reactorId`

      * 기능: 연결이 있는 [Reactor](/learn?id=reactor선) 선`ID`，현재 아무 영향도 없으며 API 호환성을 유지하기 위한 것임
      * 기본값: -1
      * 다른 값: 없음

    * `bool $ignoreError`

      * 기능: 에러를 무시할지 여부, `true`로 설정하면 연결이 닫혀도 연결 정보가 반환되며, `false`는 연결이 닫혀도 false가 반환됨
      * 기본값: false
      * 다른 값: 없음

  * **Note**

    * 클라이언트 인증서

      * 클라이언트 인증서는 [onConnect](/server/events?id=onconnect)에서만 얻을 수 있음
      * `x509` 형식으로, `openssl_x509_parse` 함수를 사용하여 인증서 정보를 얻을 수 있음

    * [dispatch_mode](/server/setting?id=dispatch_mode) = 1/3 구성을 사용할 때, 이 데이터 패킷 분배 전략은 상태가 없는 서비스에 사용됨을 고려하여, 연결이 끊어진 후 관련 정보는 직접 메모리에서 삭제되므로 `Server->getClientInfo`에서 관련 연결 정보를 가져올 수 없음.

  * **Return Value**

    * 호출 실패시 `false`를 반환
    * 호출 성공시 클라이언트 정보를 포함한 `array`를 반환

```php
$fd_info = $server->getClientInfo($fd);
var_dump($fd_info);

array(15) {
  ["server_port"]=>
  int(9501)
  ["server_fd"]=>
  int(4)
  ["socket_fd"]=>
  int(25)
  ["socket_type"]=>
  int(1)
  ["remote_port"]=>
  int(39136)
  ["remote_ip"]=>
  string(9) "127.0.0.1"
  ["reactor_id"]=>
  int(1)
  ["connect_time"]=>
  int(1677322106)
  ["last_time"]=>
  int(1677322106)
  ["last_recv_time"]=>
  float(1677322106.901918)
  ["last_send_time"]=>
  float(0)
  ["last_dispatch_time"]=>
  float(0)
  ["close_errno"]=>
  int(0)
  ["recv_queued_bytes"]=>
  int(78)
  ["send_queued_bytes"]=>
  int(0)
}
```

Parameter | 역할
---|---
server_port | 서버가 듣는 포트
server_fd | 서버 fd
socket_fd | 클라이언트 fd
socket_type | 소켓 유형
remote_port | 클라이언트 포트
remote_ip | 클라이언트 IP
reactor_id | 어느 Reactor 선에서 왔는지
connect_time | 클라이언트가 서버에 연결된 시간, 초 단위, master 프로세스에서 설정
last_time | 마지막으로 데이터를 수신한 시간, 초 단위, master 프로세스에서 설정
last_recv_time | 마지막으로 데이터를 수신한 시간, 초 단위, master 프로세스에서 설정
last_send_time | 마지막으로 데이터를 보낸 시간, 초 단위, master 프로세스에서 설정
last_dispatch_time | worker 프로세스가 데이터를 받은 시간
close_errno | 연결이 닫힌 때의 오류 코드, 연결이 비정상적으로 닫힌 경우, close_errno의 값은 0이 아닌 값이며 Linux 오류 정보 목록을 참조할 수 있음
recv_queued_bytes | 처리를 기다리는 데이터 양
send_queued_bytes | 전송을 기다리는 데이터 양
websocket_status | [선택 사항] WebSocket 연결 상태, 서버가 Swoole\WebSocket\Server인 경우 이 정보가 추가로 나타남
uid | [선택 사항] bind로 사용자 ID를 바인딩한 경우 이 정보가 추가로 나타남
ssl_client_cert | [선택 사항] SSL 터널 암호화를 사용하고 클라이언트가 인증서를 설정한 경우 이 정보가 추가됨
## getClientList()

현재 `Server`의 모든 클라이언트 연결을 반복하고, `Server::getClientList` 메서드는 공유 메모리를 기반으로 하며 `IOWait`가 없어 매우 빠른 속도로 반복합니다. 또한 `getClientList`는 현재 `Worker` 프로세스의 `TCP` 연결뿐만 아니라 모든 `TCP` 연결을 반환합니다. 별칭은 `Swoole\Server->connection_list()`

```php
Swoole\Server->getClientList(int $start_fd = 0, int $pageSize = 10): false|array
```

  * **Parameters**

    * `int $start_fd`

      * 기능: 시작 `fd` 지정
      * 기본값: 0
      * 다른 값: 없음

    * `int $pageSize`

      * 기능: 페이지 당 몇 개를 가져올지, 최대 `100`을 초과할 수 없음
      * 기본값: 10
      * 다른 값: 없음

  * **Return Value**

    * 호출이 성공하면 새로운 `start_fd`로 사용될 마지막 `fd`가 담긴 숫자 색인 배열이 반환됩니다. 배열은 작은 수부터 큰 수로 정렬됩니다. 실패하면 `false`가 반환됩니다.

  * **Tips**

    * 반복을 위해 [Server::$connections](/server/properties?id=connections) 이터레이터를 사용하는 것이 좋습니다.
    * `getClientList`는 `TCP` 클라이언트에만 사용할 수 있으며, `UDP` 서버는 클라이언트 정보를 직접 저장해야 합니다.
    * [SWOOLE_BASE](/learn?id=swoole_base) 모드에서는 현재 프로세스의 연결만 가져올 수 있습니다.

  * **Example**
  
```php
$start_fd = 0;
while (true) {
  $conn_list = $server->getClientList($start_fd, 10);
  if ($conn_list === false || count($conn_list) === 0) {
      echo "finish\n";
      break;
  }
  $start_fd = end($conn_list);
  var_dump($conn_list);
  foreach ($conn_list as $fd) {
      $server->send($fd, "broadcast");
  }
}
```  
## bind()

연결을 사용자가 정의한 `UID`에 바인딩하여, [dispatch_mode](/server/setting?id=dispatch_mode)=5을 설정하여 `hash`를 사용하여 고정으로 할당할 수 있습니다. 특정 `UID`의 연결이 모두 동일한 `Worker` 프로세스에 할당되도록 보장합니다.

```php
Swoole\Server->bind(int $fd, int $uid): bool
```

  * **Parameters**

    * `int $fd`

      * 기능: 연결의 `fd` 지정
      * 기본값: 없음
      * 다른 값: 없음

    * `int $uid`

      * 기능: 바인딩할 `UID`, 0이 아닌 숫자 여야 함
      * 기본값: 없음
      * 다른 값: `UID`는 최대 `4294967295`보다 작으면 안 되며, 최소 `-2147483648`보다 커야 합니다.

  * **Return Value**

    * `true`를 반환하면 작업 성공, `false`를 반환하면 작업 실패를 의미합니다.

  * **Note**

    * `$serv->getClientInfo($fd)`을 사용하여 연결이 바인딩된 `UID` 값을 확인할 수 있습니다.
    * 기본 [dispatch_mode](/server/setting?id=dispatch_mode)=2 설정에서, `Server`는 소켓 `fd`를 기준으로 연결 데이터를 여러 `Worker` 프로세스로 분배합니다. `fd`는 불안정하므로 클라이언트가 연결을 끊은 후 다시 연결하면 `fd`가 변경됩니다. 따라서 이 클라이언트의 데이터는 다른 `Worker`로 분배될 수 있습니다. `bind`를 사용하면 사용자가 정의한 `UID`에 따라 분배할 수 있습니다. 끊어진 다시 연결해도 동일한 `UID`의 `TCP` 연결 데이터는 같은 `Worker` 프로세스에 할당됩니다.

    * 시퀀스 문제

      * 클라이언트가 서버에 연결한 후 여러 패킷을 연속으로 보낼 경우 시퀀스 문제가 발생할 수 있습니다. `bind` 작업 중에 후속 패킷이 이미 `dispatch`될 수 있으며, 이러한 데이터 패킷은 여전히 현재 프로세스로 `fd`를 기준으로 할당됩니다. `bind` 이후에 수신된 새 데이터 패킷은 `UID` 기준으로 할당됩니다.
      * 따라서 `bind` 메커니즘을 사용하려면 네트워크 통신 프로토콜을 핸드셰이크 단계로 설계해야 합니다. 클라이언트 연결이 성공한 후 핸드셰이크 요청을 먼저 보내고, 서버가 `bind`한 후 응답한 후에 클라이언트가 새 요청을 보내야 합니다.

    * 다시 바인딩

      * 일부 상황에서, 비즈니스 로직이 사용자 연결을 다시 바인딩해야 하는 경우가 있습니다. 이 경우 연결을 끊고, 다시 `TCP` 연결을 설정하고 핸드셰이크한 후 새 `UID`에 바인딩할 수 있습니다.

    * 음수 `UID` 바인딩

      * 바인딩된 `UID`가 음수인 경우, 하위 수준에서 32비트 부호 없는 정수로 변환되며, PHP 레이어에서 32비트 부호 있는 정수로 변환해야 합니다. 다음과 같이 사용할 수 있습니다:

  ```php
  $uid = -10;
  $server->bind($fd, $uid);
  $bindUid = $server->connection_info($fd)['uid'];
  $bindUid = $bindUid >> 31 ? (~($bindUid - 1) & 0xFFFFFFFF) * -1 : $bindUid;
  var_dump($bindUid === $uid);
  ```

  * **Caution**

!> - `dispatch_mode=5`로 설정되어 있을 때만 유효합니다  
- `UID`가 바인딩되지 않은 경우 기본적으로 `fd`를 기준으로 모듈러 연산자를 사용하여 할당됩니다  
- 동일한 연결은 한 번만 `bind`할 수 있으며, 이미 `UID`가 바인딩된 경우에 다시 `bind`를 호출하면 `false`를 반환합니다

  * **Example**

```php
$serv = new Swoole\Server('0.0.0.0', 9501);

$serv->fdlist = [];

$serv->set([
    'worker_num' => 4,
    'dispatch_mode' => 5,   //uid dispatch
]);

$serv->on('connect', function ($serv, $fd, $reactor_id) {
    echo "{$fd} connect, worker:" . $serv->worker_id . PHP_EOL;
});

$serv->on('receive', function (Swoole\Server $serv, $fd, $reactor_id, $data) {
    $conn = $serv->connection_info($fd);
    print_r($conn);
    echo "worker_id: " . $serv->worker_id . PHP_EOL;
    if (empty($conn['uid'])) {
        $uid = $fd + 1;
        if ($serv->bind($fd, $uid)) {
            $serv->send($fd, "bind {$uid} success");
        }
    } else {
        if (!isset($serv->fdlist[$fd])) {
            $serv->fdlist[$fd] = $conn['uid'];
        }
        print_r($serv->fdlist);
        foreach ($serv->fdlist as $_fd => $uid) {
            $serv->send($_fd, "{$fd} say:" . $data);
        }
    }
});

$serv->on('close', function ($serv, $fd, $reactor_id) {
    echo "{$fd} Close". PHP_EOL;
    unset($serv->fdlist[$fd]);
});

$serv->start();
```
## stats()

현재 `Server`의 활성 `TCP` 연결 수, 시작 시간 등의 정보 및 `accept/close` (연결 설정/연결 종료) 총 횟수 등을 가져옵니다.

```php
Swoole\Server->stats(): array
```

  * **예시**

```php
array(25) {
  ["start_time"]=>
  int(1677310656)
  ["connection_num"]=>
  int(1)
  ["abort_count"]=>
  int(0)
  ["accept_count"]=>
  int(1)
  ["close_count"]=>
  int(0)
  ["worker_num"]=>
  int(2)
  ["task_worker_num"]=>
  int(4)
  ["user_worker_num"]=>
  int(0)
  ["idle_worker_num"]=>
  int(1)
  ["dispatch_count"]=>
  int(1)
  ["request_count"]=>
  int(0)
  ["response_count"]=>
  int(1)
  ["total_recv_bytes"]=>
  int(78)
  ["total_send_bytes"]=>
  int(165)
  ["pipe_packet_msg_id"]=>
  int(3)
  ["session_round"]=>
  int(1)
  ["min_fd"]=>
  int(4)
  ["max_fd"]=>
  int(25)
  ["worker_request_count"]=>
  int(0)
  ["worker_response_count"]=>
  int(1)
  ["worker_dispatch_count"]=>
  int(1)
  ["task_idle_worker_num"]=>
  int(4)
  ["tasking_num"]=>
  int(0)
  ["coroutine_num"]=>
  int(1)
  ["coroutine_peek_num"]=>
  int(1)
  ["task_queue_num"]=>
  int(1)
  ["task_queue_bytes"]=>
  int(1)
}
```

매개변수 | 기능
---|---
start_time | 서버가 시작된 시간
connection_num | 현재 연결 수
abort_count | 거부된 연결 수
accept_count | 수락된 연결 수
close_count | 닫힌 연결 수
worker_num  | worker 프로세스의 수
task_worker_num  | task worker 프로세스의 수【`v4.5.7`에서 사용 가능】
user_worker_num  | user worker 프로세스의 수
idle_worker_num | idle한 worker 프로세스 수
dispatch_count | Server에서 Worker로 보낸 패킷 수【`v4.5.7`에서 사용 가능, 오직 [SWOOLE_PROCESS](/learn?id=swoole_process) 모드에서만 유효】
request_count | Server가 받은 요청 수【onReceive, onMessage, onRequset, onPacket 네 종류의 데이터 요청 만을 계산한 request_count】
response_count | Server가 반환한 응답 수
total_recv_bytes| 총 수신 데이터 양
total_send_bytes | 총 송신 데이터 양
pipe_packet_msg_id | 프로세스 간 통신 id
session_round | 시작 세션 id
min_fd | 가장 작은 연결 fd
max_fd | 가장 큰 연결 fd
worker_request_count | 현재 Worker 프로세스가 받은 요청 수【worker_request_count가 max_request를 초과하면 worker 프로세스가 종료됨】
worker_response_count | 현재 Worker 프로세스 응답 수
worker_dispatch_count | master 프로세스가 현재 Worker 프로세스에 작업을 할당한 횟수, [master 프로세스](/learn?id=reactor_thread)가 dispatch할 때마다 증가
task_idle_worker_num | idle한 task 프로세스 수
tasking_num | 작업 중인 task 프로세스 수
coroutine_num | 현재 코루틴 수【Coroutine에 사용됨】, 더 많은 정보를 얻으려면 [이 섹션](/coroutine/gdb)을 참조하세요
coroutine_peek_num | 모든 코루틴 수
task_queue_num | 메시지 큐에 있는 task 수【Task에 사용됨】
task_queue_bytes | 메시지 큐의 메모리 사용량 (바이트)【Task에 사용됨】
## taskwait()

`taskwait`函数과 `task` 메서드는 비슷한 역할을 합니다. 이 함수는 비동기 작업을 [Task Worker](/learn?id=taskworker-process) 풀에 전송합니다. 그러나 `taskwait`는 작업이 완료되거나 시간 제한에 도달할 때까지 동기적으로 대기합니다. `$result`는 작업의 결과이며, `$server->finish` 함수를 통해 전송됩니다. 이 작업이 시간 초과되면 `false`가 반환됩니다.

```php
Swoole\Server->taskwait(mixed $data, float $timeout = 0.5, int $dstWorkerId = -1): mixed
```

  * **Parameters**

    * `mixed $data`

      * 기능: 전송할 작업 데이터, 어떤 유형이든 가능하며 문자열이 아닌 유형은 자동 직렬화됨
      * 기본값: 없음
      * 다른 값: 없음

    * `float $timeout`

      * 기능: 타임아웃 시간, 부동 소수점, 초 단위, 최소 `1ms` 단위 지원, 지정된 시간 내 [Task Worker](/learn?id=taskworker-process)에서 데이터를 반환하지 않으면 `taskwait`가 `false`를 반환하고 후속 작업 결과 데이터를 처리하지 않음
      * 기본값: 0.5
      * 다른 값: 없음

    * `int $dstWorkerId`

      * 기능: 어떤 [Task Worker](/learn?id=taskworker-process)에 작업을 보낼지 지정, Task Worker의 `ID`를 전달하면 됨, 범위는 `[0, $server->setting['task_worker_num']-1]`
      * 기본값: -1【기본값은 `-1`로, 무작위 전송을 의미하며, 하위 레벨에서는 빈 [Task Worker](/learn?id=taskworker-process)를 자동으로 선택함】
      * 다른 값: `[0, $server->setting['task_worker_num']-1]`

  *  **Return Value**

      * false가 반환되면 전송 실패
      * `onTask` 이벤트에서 `finish` 메소드를 실행하거나 `return`한 경우, `taskwait`는 `onTask`에 전송된 결과를 반환합니다.

  * **Tip**

    * **Coroutine Mode**

      * 버전 `4.0.4`부터 `taskwait` 메서드가 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 지원합니다. 코루틴에서 `Server->taskwait()`를 호출하면 자동으로 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)이 수행되어 더 이상 차단되지 않음.
      * [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)기능을 통해 `taskwait`를 동시에 호출할 수 있음.
      * `onTask` 이벤트에는 `finish` 메소드 또는 `return`이 하나만 존재해야 하며, 그렇지 않으면 추가로 반환되거나 `Server->finish`가 실행된 후에 `task[1] has expired` 경고가 표시될 수 있음.

    * **Synchronous Mode**

      * 동기 블로킹 모드에서, `taskwait`는 [Unix Socket](/learn?id=ipc) 통신 및 공유 메모리를 사용하여 데이터를 `Worker` 프로세스에 반환하는 동기 블로킹 프로세스임.

    * **Special Case**

      * [onTask](/server/events?id=ontask)에서 어떤 [동기 I/O](/learn?id=동기io-비동기io) 작업도 없는 경우, 기본적으로 프로세스 전환이 두 번만 발생하며 `IO` 대기가 발생하지 않음, 따라서 이 경우 `taskwait`는 블로킹이 아닌 것으로 간주됨. 실제로 [onTask](/server/events?id=ontask)에서는 `PHP` 배열을 읽거나 쓰기만 하는 것으로 `10`만 번의 `taskwait` 작업을 수행하더라도 총 소요 시간은 `1`초만 소요되며, 평균적으로 각 작업당 `10`마이크로초 소요됨

  * **Note**

  !> - `Swoole\Server::finish` 사용하지 말 것  
- `taskwait` 메서드는 [Task Worker](/learn?id=taskworker-process)에서 호출할 수 없음.
## taskWaitMulti()

여러 `task` 비동기 작업을 동시에 실행하고, 이 메서드는 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 지원하지 않으며, 다른 코루틴이 시작되는 것을 유발할 수 있습니다. 코루틴 환경에서는 이후에 설명되는 `taskCo`를 사용해야 합니다.

```php
Swoole\Server->taskWaitMulti(array $tasks, float $timeout = 0.5): false|array
```

  * **매개변수**

    * `array $tasks`

      * 설명: 숫자 인덱스 배열이어야 하며 연관된 인덱스 배열은 지원되지 않음. 내부적으로 `$tasks`를 반복하여 각 작업을 [Task 프로세스](/learn?id=taskworker-프로세스)에 전달함
      * 기본값: 없음
      * 다른 값: 없음

    * `float $timeout`

      * 설명: 부동 소수점 숫자이며 단위는 초
      * 기본값: 0.5초
      * 다른 값: 없음

  * **반환 값**

    * 모든 작업이 완료되거나 시간 초과되면 결과 배열을 반환함. 결과 배열의 각 작업 결과는 `$tasks`와 해당 색인에 대응됨. 즉, `$tasks[2]`에 대응하는 결과는 `$result[2]`임
    * 특정 작업이 시간 초과되어도 다른 작업에 영향을 주지 않으며, 반환된 결과 데이터에는 시간 초과된 작업이 포함되지 않음

  * **주의사항**

  !> -최대 동시 작업 수는 `1024`를 초과할 수 없음

  * **예시**

```php
$tasks[] = mt_rand(1000, 9999); // Task 1
$tasks[] = mt_rand(1000, 9999); // Task 2
$tasks[] = mt_rand(1000, 9999); // Task 3
var_dump($tasks);

// 모든 Task 결과가 반환될 때까지 대기, 제한 시간은 10초
$results = $server->taskWaitMulti($tasks, 10.0);

if (!isset($results[0])) {
    echo "Task 1이 시간 초과되었습니다\n";
}
if (isset($results[1])) {
    echo "Task 2의 실행 결과는 {$results[1]}\n";
}
if (isset($results[2])) {
    echo "Task 3의 실행 결과는 {$results[2]}\n";
}
```
## taskCo()

`Task`를 동시에 실행하고 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)을 수행하여 `taskWaitMulti` 기능을 지원합니다.

```php
Swoole\Server->taskCo(array $tasks, float $timeout = 0.5): false|array
```

* `$tasks`: 작업 목록으로, 배열이어야 합니다. 내부적으로 배열을 순회하여 각 요소를 `Task` 프로세스 풀에 작업으로 보냅니다.
* `$timeout`: 기본값은 `0.5`초이며, 지정된 시간 내에 작업이 모두 완료되지 않으면 즉시 중단되고 결과가 반환됩니다.
* 작업이 완료되거나 시간이 초과되면 결과 배열이 반환됩니다. 결과 배열의 각 작업 결과는 `$tasks`와 일치합니다. 예를 들어, `$tasks[2]`의 결과는 `$result[2]`입니다.
* 어떤 작업이 실패하거나 시간 초과되면 해당 결과 배열 항목은 `false`가 됩니다. 즉, `$tasks[2]`가 실패하면 `$result[2]`의 값은 `false`가 됩니다.

!> 최대 동시 작업 수는 `1024`여야 합니다.

  * **스케줄링 과정**

    * `$tasks` 목록의 각 작업은 무작위로 `Task` 작업 프로세스에 전달되고, 전달이 완료되면 현재 코루틴을 양보하고 `$timeout` 초의 타이머를 설정합니다.
    * `onFinish`에서 해당 작업 결과를 수집하여 결과 배열에 저장합니다. 모든 작업이 결과를 반환했는지 확인하고, 아닌 경우 계속 대기합니다. 모든 작업이 결과를 반환하면 해당 코루틴의 실행을 `resume`하고, 시간 초과 타이머를 제거합니다.
    * 지정된 시간 내에 작업이 모두 완료되지 않는 경우, 타이머가 먼저 트리거되어 대기 상태가 초기화됩니다. 완료되지 않은 작업 결과가 `false`로 표시되고 해당 코루틴이 즉시 `resume`됩니다.

  * **예시**

```php
$server = new Swoole\Http\Server("127.0.0.1", 9502, SWOOLE_BASE);

$server->set([
    'worker_num'      => 1,
    'task_worker_num' => 2,
]);

$server->on('Task', function (Swoole\Server $serv, $task_id, $worker_id, $data) {
    echo "#{$serv->worker_id}\tonTask: worker_id={$worker_id}, task_id=$task_id\n";
    if ($serv->worker_id == 1) {
        sleep(1);
    }
    return $data;
});

$server->on('Request', function ($request, $response) use ($server) {
    $tasks[0] = "hello world";
    $tasks[1] = ['data' => 1234, 'code' => 200];
    $result   = $server->taskCo($tasks, 0.5);
    $response->end('Test End, Result: ' . var_export($result, true));
});

$server->start();
```
## finish()

[Task Worker process](/learn?id=taskworker进程)에서 작업이 완료되었음을 `Worker` 프로세스에 알리는 데 사용됩니다. 이 함수는 결과 데이터를 `Worker` 프로세스로 전달할 수 있습니다.

```php
Swoole\Server->finish(mixed $data): bool
```

  * **Parameters**

    * `mixed $data`

      * 기능: 작업 처리 결과 내용
      * 기본값: 없음
      * 다른 값: 없음

  * **Return Value**

    * `true`를 반환하면 작업이 성공적으로 완료되었음을 나타냅니다. `false`를 반환하면 작업이 실패했음을 나타냅니다.

  * **Tips**
    * `finish` 메소드는 여러 번 호출할 수 있으며, `Worker` 프로세스는 여러 번 [onFinish](/server/events?id=onfinish) 이벤트를 트리거합니다.
    * [onTask](/server/events?id=ontask) 콜백 함수에서 `finish` 메소드를 호출한 후에도, `return` 데이터는 여전히 [onFinish](/server/events?id=onfinish) 이벤트를 트리거합니다.
    * `Server->finish`는 선택사항입니다. 만약 `Worker` 프로세스가 작업 실행 결과에 관심이 없다면, 이 함수를 호출할 필요가 없습니다.
    * [onTask](/server/events?id=ontask) 콜백 함수에서 `return` 문자열을 반환하는 것은 `finish`를 호출하는 것과 동일합니다.

  * **Note**

  !> `Server->finish` 함수를 사용하기 위해서는 `Server`에 [onFinish](/server/events?id=onfinish) 콜백 함수를 설정해야 합니다. 이 함수는 [Task Worker process](/learn?id=taskworker进程)의 [onTask](/server/events?id=ontask) 콜백 함수에서만 사용할 수 있습니다.
## heartbeat()

[heartbeat_check_interval](/server/setting?id=heartbeat_check_interval)과는 달리, 이 메서드는 모든 서버 연결을 자체적으로 검사하고 약속된 시간을 초과한 연결을 찾습니다. `if_close_connection`을 지정하면 자동으로 시간 초과된 연결을 닫습니다. 지정하지 않으면 연결의 `fd` 배열만 반환합니다.

```php
Swoole\Server->heartbeat(bool $ifCloseConnection = true): bool|array
```

  * **Parameters**

    * `bool $ifCloseConnection`

      * 기능: 시간 초과된 연결을 닫을지 여부
      * 기본값: true
      * 다른 값: false

  * **Return Value**

    * 성공하면 닫힌 `$fd` 요소로 이루어진 연속 배열을 반환합니다.
    * 실패하면 `false`를 반환합니다.

  * **Example**

```php
$closeFdArray = $server->heartbeat();
```
## getLastError()

가장 최근에 발생한 작업 오류 코드를 얻습니다. 비즈니스 코드에서는 오류 코드 유형에 따라 다른 논리를 실행할 수 있습니다.

```php
Swoole\Server->getLastError(): int
```

  * **리턴 값**

오류 코드 | 설명
---|---
1001 | 해당 연결은 이미 `Server` 측에서 닫혀 있으며, 일반적으로 이 오류는 코드에서 이미 `$server->close()`를 실행하여 특정 연결을 닫았지만 여전히 이 연결로 데이터를 전송하려면 `$server->send()`를 호출하는 경우에 발생합니다.
1002 | 해당 연결은 `Client` 측에서 이미 닫혔으며, `Socket`이 닫혀 있어 상대방에게 데이터를 전송할 수 없습니다.
1003 | `close`를 실행 중이며, [onClose](/server/events?id=onclose) 콜백 함수에서는 `$server->send()`를 사용할 수 없습니다.
1004 | 연결이 이미 닫혔습니다.
1005 | 연결이 존재하지 않으며, 전달된 `$fd`가 잘못되었을 수 있습니다.
1007 | 시간 초과된 데이터를 수신하여, `TCP`가 연결을 닫은 후, 일부 데이터가 [UnixSocket](/learn?id=ipc) 캐시 영역에 남아 있을 수 있으며, 이러한 데이터는 삭제됩니다.
1008 | 전송 버퍼가 가득 찼기 때문에 `send` 작업을 실행할 수 없습니다. 이 오류가 발생하면 이 연결의 상대방이 즉각적으로 데이터를 수신하지 못하여 전송 버퍼가 가득 차 있음을 의미합니다.
1202 | 전송한 데이터가 [server->buffer_output_size](/server/setting?id=buffer_output_size) 설정을 초과했습니다.
9007 | [dispatch_mode](/server/setting?id=dispatch_mode)=3을 사용할 때만 발생하며, 현재 사용 가능한 프로세스가 없음을 나타내며, `worker_num` 프로세스 수를 늘릴 수 있습니다.
## getSocket()

이 메서드를 호출하면 하위 `socket` 핸들을 얻을 수 있으며 반환되는 객체는 `sockets` 리소스 핸들입니다.

```php
Swoole\Server->getSocket(): false|\Socket
```

!> 이 메서드는 PHP의 `sockets` 확장 기능에 의존하며 `Swoole` 를 컴파일 할 때 `--enable-sockets` 옵션을 활성화해야 합니다.

  * **포트 수신**

    * `listen` 메서드로 추가한 포트를 사용하여 `Swoole\Server\Port` 객체가 제공하는 `getSocket` 메서드를 사용할 수 있습니다.

    ```php
    $port = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $socket = $port->getSocket();
    ```

    * `socket_set_option` 함수를 사용하여 더 낮은 수준의 `socket` 매개변수를 설정할 수 있습니다.

    ```php
    $socket = $server->getSocket();
    if (!socket_set_option($socket, SOL_SOCKET, SO_REUSEADDR, 1)) {
        echo 'Unable to set option on socket: '. socket_strerror(socket_last_error()) . PHP_EOL;
    }
    ```

  * **멀티캐스트 지원**

    * `socket_set_option`을 사용하여 `MCAST_JOIN_GROUP` 매개변수를 설정하면 `Socket`을 멀티캐스트에 가입시켜 네트워크 멀티캐스트 데이터 패킷을 수신할 수 있습니다.

```php
$server = new Swoole\Server('0.0.0.0', 9905, SWOOLE_BASE, SWOOLE_SOCK_UDP);
$server->set(['worker_num' => 1]);
$socket = $server->getSocket();

$ret = socket_set_option(
    $socket,
    IPPROTO_IP,
    MCAST_JOIN_GROUP,
    array(
        'group' => '224.10.20.30', // 멀티캐스트 주소를 나타냅니다
        'interface' => 'eth0' // 네트워크 인터페이스의 이름을 나타내며 숫자 또는 문자열 일 수 있습니다. 예: eth0, wlan0
    )
);

if ($ret === false) {
    throw new RuntimeException('Unable to join multicast group');
}

$server->on('Packet', function (Swoole\Server $server, $data, $addr) {
    $server->sendto($addr['address'], $addr['port'], "Swoole: $data");
    var_dump($addr, strlen($data));
});

$server->start();
```
## protect()

클라이언트 연결을 보호 상태로 설정하여 하트비트 스레드에 의해 잘려지지 않습니다.

```php
Swoole\Server->protect(int $fd, bool $is_protected = true): bool
```

  * **매개변수**

    * `int $fd`

      * 기능: 클라이언트 연결 `fd` 지정
      * 기본값: 없음
      * 다른 값: 없음

    * `bool $is_protected`

      * 기능: 설정된 상태
      * 기본값: true 【보호 상태임을 나타냄】
      * 다른 값: false 【보호하지 않음을 나타냄】

  * **반환 값**

    * `true`를 반환하면 작업이 성공적으로 완료된 것이며, `false`를 반환하면 작업에 실패한 것입니다.
## confirm()

연결을 확인하고 [enable_delay_receive](/server/setting?id=enable_delay_receive)과 함께 사용합니다. 클라이언트가 연결을 설정한 후에도 읽기 이벤트를 청취하지 않고, 단순히 [onConnect](/server/events?id=onconnect) 이벤트 콜백을 실행시키며, [onConnect](/server/events?id=onconnect) 콜백에서 `confirm`을 실행하여 연결을 확인할 때에 서버가 읽기 이벤트를 청취하고 클라이언트에서 오는 데이터를 수신합니다.

!> Swoole 버전 >= `v4.5.0`에서 사용 가능

```php
Swoole\Server->confirm(int $fd): bool
```

  * **매개변수**

    * `int $fd`

      * 기능: 연결의 고유 식별자
      * 기본값: 없음
      * 다른 값: 없음

  * **반환 값**
  
    * 확인이 성공하면 `true`가 반환됨
    * `$fd`에 해당하는 연결이 존재하지 않거나 이미 닫혔거나 이미 청취 상태인 경우, 실패한 것으로 간주하고 `false`를 반환함

  * **용도**
  
    이 메소드는 일반적으로 서버를 보호하기 위해 사용되며, 과부하 공격을 방지합니다. 클라이언트 연결을 받았을 때 [onConnect](/server/events?id=onconnect) 함수가 트리거되며, 클라이언트의 `IP` 출처를 확인하고 서버에 데이터를 보낼 수 있는지 여부를 판단할 수 있습니다.

  * **예시**
    
```php
//Server 객체 생성, 127.0.0.1:9501 포트를 수신대기
$serv = new Swoole\Server("127.0.0.1", 9501); 
$serv->set([
    'enable_delay_receive' => true,
]);

//연결 이벤트 수신 대기
$serv->on('Connect', function ($serv, $fd) {  
    //여기서 $fd를 확인하고 문제가 없으면 confirm 실행
    $serv->confirm($fd);
});

//데이터 수신 이벤트 수신 대기
$serv->on('Receive', function ($serv, $fd, $reactor_id, $data) {
    $serv->send($fd, "Server: ".$data);
});

//연결 닫힘 이벤트 수신 대기
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

//서버 시작
$serv->start(); 
```
## getWorkerId()

현재 `Worker` 프로세스의 `id`를 가져옵니다(프로세스의 `PID`가 아님), [onWorkerStart](/server/events?id=onworkerstart)에서의 `$workerId`와 일치합니다.

```php
Swoole\Server->getWorkerId(): int|false
```

!> Swoole 버전 >= `v4.5.0RC1`에서 사용 가능합니다.
## getWorkerPid()

지정된 `Worker` 프로세스의 `PID`를 가져옵니다.

```php
Swoole\Server->getWorkerPid(int $worker_id = -1): int|false
```

  * **매개변수**

    * `int $worker_id`

      * 기능: 지정된 프로세스의 PID를 가져옵니다.
      * 기본값: -1, [-1은 현재 프로세스를 의미합니다.]
      * 다른 값: 없음

!> Swoole 버전 >= `v4.5.0RC1`에서 사용 가능
## getWorkerStatus()

`Worker` 프로세스 상태를 가져옵니다.

```php
Swoole\Server->getWorkerStatus(int $worker_id = -1): int|false
```

!> Swoole 버전 >= `v4.5.0RC1`에서 사용 가능합니다.

  * **매개변수**

    * `int $worker_id`

      * 기능: 프로세스 상태를 가져옵니다.
      * 기본값: -1, [현재 프로세스]를 나타냅니다.
      * 다른 값: 없음

  * **반환 값**
  
    * `Worker` 프로세스 상태를 반환하며, 프로세스 상태 값 참조
    * `Worker` 프로세스가 아니거나 프로세스가 없는 경우 `false`를 반환합니다.

  * **프로세스 상태 값**

    상수 | 값 | 설명 | 버전 의존
    ---|---|---|---
    SWOOLE_WORKER_BUSY | 1 | 바쁨 | v4.5.0RC1
    SWOOLE_WORKER_IDLE | 2 | 유휴 | v4.5.0RC1
    SWOOLE_WORKER_EXIT | 3 | [reload_async](/server/setting?id=reload_async)를 활성화하면, 동일한 worker_id에 두 개의 프로세스가 있을 수 있습니다. 새로운 프로세스와 이전 프로세스 중, 이전 프로세스가 읽는 상태 코드는 EXIT입니다. | v4.5.5
## getManagerPid()

현재 서비스의 `Manager` 프로세스 `PID`를 가져옵니다.

```php
Swoole\Server->getManagerPid(): int
```

!> Swoole 버전 >= `v4.5.0RC1`에서 사용 가능
## getMasterPid()

현재 서비스의 `Master` 프로세스 `PID`를 가져옵니다.

```php
Swoole\Server->getMasterPid(): int
```

!> Swoole 버전 >= `v4.5.0RC1`에서 사용 가능합니다
## addCommand()

```php
Swoole\Server->addCommand(string $name, int $accepted_process_types, Callable $callback): bool
```

!> -Swoole版本 >= `v4.8.0` 可用         
  -该函数只能在服务未启动前调用，存在同名命令的话会直接返回`false`

* **매개변수**

    * `string $name`

        * 기능: `command`의 이름
        * 기본값: 없음
        * 다른 값: 없음

    * `int $accepted_process_types`

      * 기능: 요청을 수락할 프로세스 유형, 여러 프로세스 유형을 지원하려면 `|`를 사용하여 연결할 수 있음. 예를 들어 `SWOOLE_SERVER_COMMAND_MASTER | SWOOLE_SERVER_COMMAND_MANAGER`
      * 기본값: 없음
      * 다른 값:
        * `SWOOLE_SERVER_COMMAND_MASTER`: 마스터 프로세스
        * `SWOOLE_SERVER_COMMAND_MANAGER`: 매니저 프로세스
        * `SWOOLE_SERVER_COMMAND_EVENT_WORKER`: 워커 프로세스
        * `SWOOLE_SERVER_COMMAND_TASK_WORKER`: 작업 프로세스

    * `callable $callback`

        * 기능: 콜백 함수. 두 개의 인수를 가지며, 하나는 `Swoole\Server` 클래스이고, 다른 하나는 사용자 정의 변수입니다. 이 변수는 `Swoole\Server::command()`의 4번째 매개변수를 통해 전달됩니다.
        * 기본값: 없음
        * 다른 값: 없음

* **반환 값**

    * `true`를 반환하면 사용자 정의 명령어 추가 성공, `false`를 반환하면 실패
## command()

정의된 사용자 정의 명령어 `command`를 호출합니다.

```php
Swoole\Server->command(string $name, int $process_id, int $process_type, mixed $data, bool $json_decode = true): false|string|array
```

!>Swoole 버전 >= `v4.8.0`에서 사용 가능하며, `SWOOLE_PROCESS` 및 `SWOOLE_BASE` 모드에서 이 함수는 `master` 프로세스에서만 사용할 수 있습니다.

* **매개변수**

    * `string $name`

        * 기능: `command` 이름
        * 기본값: 없음
        * 다른 값: 없음

    * `int $process_id`

        * 기능: 프로세스 ID
        * 기본값: 없음
        * 다른 값: 없음

    * `int $process_type`

        * 기능: 프로세스 요청 유형, 아래의 다른 값 중 하나만 선택해야 합니다.
        * 기본값: 없음
        * 다른 값:
          * `SWOOLE_SERVER_COMMAND_MASTER` master 프로세스
          * `SWOOLE_SERVER_COMMAND_MANAGER` manager 프로세스
          * `SWOOLE_SERVER_COMMAND_EVENT_WORKER` worker 프로세스
          * `SWOOLE_SERVER_COMMAND_TASK_WORKER` task 프로세스

    * `mixed $data`

        * 기능: 요청 데이터, 해당 데이터는 직렬화할 수 있어야 합니다.
        * 기본값: 없음
        * 다른 값: 없음

    * `bool $json_decode`

        * 기능: `json_decode`를 사용하여 해석할지 여부
        * 기본값: true
        * 다른 값: false
  
  * **사용 예시**
    ```php
    <?php
    use Swoole\Http\Server;
    use Swoole\Http\Request;
    use Swoole\Http\Response;

    $server = new Server('127.0.0.1', 9501, SWOOLE_BASE);
    $server->addCommand('test_getpid', SWOOLE_SERVER_COMMAND_MASTER | SWOOLE_SERVER_COMMAND_EVENT_WORKER,
        function ($server, $data) {
          var_dump($data);
          return json_encode(['pid' => posix_getpid()]);
        });
    $server->set([
        'log_file' => '/dev/null',
        'worker_num' => 2,
    ]);

    $server->on('start', function (Server $serv) {
        $result = $serv->command('test_getpid', 0, SWOOLE_SERVER_COMMAND_MASTER, ['type' => 'master']);
        Assert::eq($result['pid'], $serv->getMasterPid());
        $result = $serv->command('test_getpid', 1, SWOOLE_SERVER_COMMAND_EVENT_WORKER, ['type' => 'worker']);
        Assert::eq($result['pid'], $serv->getWorkerPid(1));
        $result = $serv->command('test_not_found', 1, SWOOLE_SERVER_COMMAND_EVENT_WORKER, ['type' => 'worker']);
        Assert::false($result);

        $serv->shutdown();
    });

    $server->on('request', function (Request $request, Response $response) {
    });
    $server->start();
    ```
