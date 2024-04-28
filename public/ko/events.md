# 이벤트

이 섹션에서는 모든 Swoole 콜백 함수를 소개하며, 각 콜백 함수는 PHP 함수에 해당하며 이벤트와 대응됩니다.
## onStart

?> **마스터 프로세스의 주 스레드에서 이 함수를 콜백합니다**

```php
function onStart(Swoole\Server $server);
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본값**：없음
      * **기타 값**：없음

* **이벤트 이전에 `Server`에서 수행한 작업들**

    * 매니저 프로세스 생성이 완료됨 [Manager 프로세스](/learn?id=manager프로세스)
    * 워커 서브 프로세스가 생성되었습니다 [Worker 프로세스](/learn?id=worker프로세스)
    * 모든 TCP/UDP/[unixSocket](/learn?id=ipc란) 포트를 수신 대기하고 있지만 연결 및 요청 수락은 시작되지 않았습니다
    * 타이머가 등록되었습니다

* **다음 작업**

    * 주 [리액터](/learn?id=리액터 스레드)가 이벤트를 수신하기 시작하고 클라이언트가 `Server`에 `connect` 할 수 있게 됩니다

**`onStart` 콜백에서는 `echo`, `Log` 출력, 프로세스 이름을 수정하는 것만 허용됩니다. 다른 작업을 수행해서는 안 됩니다(`server` 관련 함수를 호출하는 것과 같은 작업은 허용되지 않음) 서비스가 준비되지 않았기 때문입니다. `onWorkerStart`와 `onStart` 콜백은 서로 다른 프로세스에서 병렬로 실행되며 순서는 없습니다.**

`onStart` 콜백에서는 `$server->master_pid`와 `$server->manager_pid` 값을 파일에 저장할 수 있습니다. 이렇게 하면 이 두 `PID`에 신호를 보내서 종료 및 재시작 작업을 수행할 수 있는 스크립트를 작성할 수 있습니다.

`onStart` 이벤트는 `Master` 프로세스의 주 스레드에서 호출됩니다.

!> `onStart`에서 생성된 전역 리소스 객체는 `Worker` 프로세스에서 사용할 수 없습니다. 왜냐하면 `onStart` 호출 시에 이미 `worker` 프로세스가 생성되어 있기 때문입니다.  
새로 생성된 객체는 주 프로세스에서 만들어지며 `worker` 프로세스는 이 메모리 영역에 액세스할 수 없습니다.  
따라서 전역 객체 생성 코드는 `Server::start` 이전에 배치되어야 하며, 대표적인 예시는 [Swoole\Table](/memory/table?id=complete-example)입니다.

* **보안 팁**

`onStart` 콜백에서 비동기 및 코루틴 API를 사용할 수 있지만, 이는 `dispatch_func`와 `package_length_func`과 충돌할 수 있으므로 **동시에 사용하지 마십시오**.

`onStart`에서 타이머를 시작하지 마십시오. 코드에서 `Swoole\Server::shutdown()`을 실행하면 항상 타이머가 실행 중이기 때문에 프로그램이 종료되지 않을 수 있습니다.

`onStart` 콜백에서 `return` 이전에 서버 프로그램은 어떠한 클라이언트 연결도 수락하지 않기 때문에 동기 블로킹 함수를 안전하게 사용할 수 있습니다.

* **BASE 모드**

[SWOOLE_BASE](/learn?id=swoole_base) 모드에서는 `master` 프로세스가 없으므로 `onStart` 이벤트가 없습니다. `BASE` 모드에서는 `onStart` 콜백 함수를 사용하지 마십시오.

```
경고 swReactorProcess_start: SWOOLE_BASE와 함께 onStart 이벤트 사용이 중단되었습니다
```
## onBeforeShutdown

?> **이 이벤트는 `Server`가 정상적으로 종료되기 전에 발생합니다.**

!> Swoole 버전 >= `v4.8.0`에서 사용할 수 있습니다. 이 이벤트에서는 코루틴 API를 사용할 수 있습니다.

```php
function onBeforeShutdown(Swoole\Server $server);
```


* **파라미터**

    * **`Swoole\Server $server`**
        * **기능**: Swoole\Server 객체
        * **기본값**: 없음
        * **다른 값**: 없음
## onShutdown

?> **이 이벤트는`Server`가 정상적으로 종료될 때 발생합니다.**

```php
function onShutdown(Swoole\Server $server);
```

  * **매개변수**

    * **`Swoole\Server $server`**
      * **기능**: Swoole\Server 객체
      * **기본값**: 없음
      * **기타값**: 없음

  * **이전에`Swoole\Server`가 수행한 작업**

    * 모든 [Reactor](/learn?id=reactor선) 스레드, `HeartbeatCheck` 스레드, `UdpRecv` 스레드가 종료되었습니다.
    * 모든 `Worker` 프로세스, [Task 프로세스](/learn?id=taskworker프로세스), [User 프로세스](/server/methods?id=addprocess)가 종료되었습니다.
    * 모든 `TCP/UDP/UnixSocket` 수신 포트가 `close`되었습니다.
    * 주 [Reactor](/learn?id=reactor선) 가 종료되었습니다.

  !> 강제로 `kill`된 프로세스는 `onShutdown`이 호출되지 않습니다. 예: `kill -9`  
  정상적인 종료를 위해 주 프로세스로 `SIGTERM` 신호를 보내기 위해 `kill -15`를 사용해야 합니다.  
  명령줄에서 `Ctrl+C`를 사용하여 프로그램을 중단하면 즉시 중지되며, 하위 레이어에서 `onShutdown`이 호출되지 않습니다.

  * **주의사항**

  !> `onShutdown`에서는 비동기 또는 코루틴 관련 `API`를 호출하지 마십시오.  
`onShutdown`이 트리거될 때 이미 모든 이벤트 루프 시설이 해제되었습니다.  
이 시점에서는 더 이상 코루틴 환경이 존재하지 않으며, 개발자가 코루틴 관련 `API`를 사용해야 하는 경우 수동으로 `Co\run`을 호출하여 [코루틴 컨테이너](/coroutine?id=코루틴_컨테이너란)를 만들어야 합니다.
## onWorkerStart

?> **이 이벤트는 Worker 프로세스/ [Task 프로세스](/learn?id=taskworker프로세스)가 시작될 때 발생하며, 여기에서 생성된 객체는 프로세스 수명 주기 동안 사용할 수 있습니다.**


```php
function onWorkerStart(Swoole\Server $server, int $workerId);
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $workerId`**
      * **기능**：`Worker` 프로세스 `id` (프로세스의 PID가 아님)
      * **기본값**：없음
      * **다른 값**：없음

  * `onWorkerStart/onStart`는 병렬로 실행되며 순서가 없습니다.
  * `$server->taskworker` 속성을 통해 현재 `Worker` 프로세스인지 [Task 프로세스](/learn?id=taskworker프로세스)인지 판단할 수 있습니다.
  * `worker_num` 및 `task_worker_num`을 `1`보다 크게 설정하면 각 프로세스마다 `onWorkerStart` 이벤트가 한 번씩 발생하며, [$worker_id](/server/properties?id=worker_id)를 통해 서로 다른 작업 프로세스를 구분할 수 있습니다.
  * `worker` 프로세스에서 `task` 프로세스로 작업을 보내고, `task` 프로세스가 모든 작업을 처리한 후 [onFinish](/server/events?id=onfinish) 콜백 함수를 통해`worker` 프로세스에 통보합니다. 예를 들어, 백그라운드 작업으로 십만 명의 사용자에게 공지 이메일을 발송하고, 작업이 완료되면 상태가 '발송 중'으로 표시됩니다. 이때 다른 작업을 계속할 수 있으며, 메일 발송이 완료되면 상태가 자동으로 '발송됨'으로 변경됩니다.

  다음 예제는 Worker 프로세스/ [Task 프로세스](/learn?id=taskworker프로세스)의 이름을 변경하는 데 사용됩니다.

```php
$server->on('WorkerStart', function ($server, $worker_id){
    global $argv;
    if($worker_id >= $server->setting['worker_num']) {
        swoole_set_process_name("php {$argv[0]} task worker");
    } else {
        swoole_set_process_name("php {$argv[0]} event worker");
    }
});
```

  코드를 다시로드하는 [Reload](/server/methods?id=reload) 메커니즘을 사용하려면 파일을 `onWorkerStart`에서 `require`해야 하며 파일 상단에 위치시키면 안 됩니다. `onWorkerStart` 호출 전에 이미 포함된 파일은 코드가 다시로드되지 않습니다.

  공유되고 변하지 않는 PHP 파일을 `onWorkerStart` 이전에 배치할 수 있습니다. 이렇게 하면 코드를 다시로드할 수는 없지만 모든 `Worker`가 공유되므로 이러한 데이터를 저장하기 위한 추가 메모리가 필요하지 않습니다.
`onWorkerStart` 이후의 코드는 각 프로세스마다 메모리에 유지해야 합니다.

  * `$worker_id`는 이 `Worker` 프로세스의 `ID`를 나타내며, 범위는 [$worker_id](/server/properties?id=worker_id)를 참조하십시오.
  * [$worker_id](/server/properties?id=worker_id)와 프로세스 `PID`는 관련이 없으며, `posix_getpid` 함수를 사용하여 `PID`를 가져올 수 있습니다.

  * **Coroutine 지원**

    * `onWorkerStart` 콜백 함수에서 자동으로 코루틴이 생성되므로 `onWorkerStart`에서 코루틴 `API`를 호출할 수 있습니다.

  * **주의**

    !> 치명적인 오류가 발생하거나 코드에서 명시적으로 `exit`를 호출하는 경우 `Worker/Task` 프로세스가 종료되고 관리 프로세스가 새로운 프로세스를 생성합니다. 이는 무한 루프를 초래하여 프로세스를 계속 생성 및 해제할 수 있습니다.
## onWorkerStop

?> **이 이벤트는`Worker` 프로세스가 종료될 때 발생합니다. 여기서는`Worker` 프로세스가 할당한 모든 리소스를 회수할 수 있습니다.**

```php
function onWorkerStop(Swoole\Server $server, int $workerId);
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**: Swoole\Server 객체
      * **기본값**: 없음
      * **다른 값**: 없음

    * **`int $workerId`**
      * **기능**: `Worker` 프로세스 `id` (프로세스 PID가 아님)
      * **기본값**: 없음
      * **다른 값**: 없음

  * **주의사항**

    !> -프로세스가 예기치 않게 종료되는 경우, 강제로 `kill`되거나 치명적 오류, `core dump`가 발생하는 경우`onWorkerStop` 콜백 함수가 실행되지 않습니다.  
    -`onWorkerStop`에서 비동기 또는 코루틴 관련 `API`를 호출하지 마십시오. `onWorkerStop`이 트리거되는 시점에는 모든 이벤트 루프 구성 요소가 이미 파괴되었습니다.
## onWorkerExit

?> **[reload_async](/server/setting?id=reload_async) 특성을 활성화했을 때에만 유효합니다. [서비스를 올바르게 다시 시작하는 방법](/question/use?id=swoole如何正确的重启服务)를 참조하세요.**

```php
function onWorkerExit(Swoole\Server $server, int $workerId);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server 객체
      * **Default**: 없음
      * **Other values**: 없음

    * **`int $workerId`**
      * **Description**: `Worker` 프로세스의 `id` (프로세스 PID가 아님)
      * **Default**: 없음
      * **Other values**: 없음

  * **Notes**

    !> - `Worker` 프로세스가 종료되지 않으면 `onWorkerExit`가 계속해서 트리거됩니다.  
    - `Worker` 프로세스 내에서 `onWorkerExit`가 발생하며, [Task 프로세스](/learn?id=taskworker进程)에 이벤트 루프가 있다면 해당 프로세스에서도 트리거됩니다.  
    - `onWorkerExit`에서 비동기 `Socket` 연결을 최대한으로 제거/닫습니다. 최종적으로 백그라운드에서 [이벤트 루프](/learn?id=什么是eventloop)가 이벤트를 모니터링하는 핸들러 수가 `0`이 되면 프로세스가 종료됩니다.  
    - 이벤트 핸들러를 감지하지 않는 경우에는 프로세스가 종료될 때 이 함수가 콜백되지 않습니다.  
    - `Worker` 프로세스가 종료된 후에야 `onWorkerStop` 이벤트 콜백이 실행됩니다.
## onConnect

?> **워커(worker) 프로세스에서 새로운 연결이 발생했을 때 호출됩니다.**

```php
function onConnect(Swoole\Server $server, int $fd, int $reactorId);
```

  * **파라미터** 

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본값**：없음
      * **기타 값**：없음

    * **`int $fd`**
      * **기능**：연결의 파일 디스크립터
      * **기본값**：없음
      * **기타 값**：없음

    * **`int $reactorId`**
      * **기능**：연결이 발생한 [리액터(Reactor)](/learn?id=reactor-쓰레드) 쓰레드의 `ID`
      * **기본값**：없음
      * **기타 값**：없음

  * **주의**

    !> `onConnect/onClose` 콜백은 `Worker` 프로세스 내에서 발생하며, 메인 프로세스에서는 일어나지 않습니다.  
    `UDP` 프로토콜을 사용하는 경우 [onReceive](/server/events?id=onreceive) 이벤트만 존재하며, `onConnect/onClose` 이벤트는 없습니다.

    * **[dispatch_mode](/server/setting?id=dispatch_mode) = 1/3**

      * 이 모드에서는 `onConnect/onReceive/onClose`가 서로 다른 프로세스로 전달될 수 있습니다. 연결 관련 `PHP` 객체 데이터는 [onConnect](/server/events?id=onconnect) 콜백에서 데이터를 초기화할 수 없으며, [onClose](/server/events?id=onclose)에서 데이터를 정리할 수 없습니다.
      * `onConnect/onReceive/onClose` 세 가지 이벤트는 동시에 실행될 수 있으며, 예외를 가져올 수 있습니다.
## onReceive

?> **`worker`프로세스에서 발생하는 데이터 수신 시에 콜백되는 함수입니다.**

```php
function onReceive(Swoole\Server $server, int $fd, int $reactorId, string $data);
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**: Swoole\Server 객체
      * **기본값**: 없음
      * **기타**: 없음

    * **`int $fd`**
      * **기능**: 연결된 파일 디스크립터
      * **기본값**: 없음
      * **기타**: 없음

    * **`int $reactorId`**
      * **기능**: `TCP` 연결이 발생한 [Reactor](/learn?id=reactor선) 쓰레드의 `ID`
      * **기본값**: 없음
      * **기타**: 없음

    * **`string $data`**
      * **기능**: 수신한 데이터 내용, 텍스트 또는 이진 콘텐츠가 될 수 있음
      * **기본값**: 없음
      * **기타**: 없음

  * **`TCP` 프로토콜의 데이터 완정성에 대한 내용은 [TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터_패킷_경계_문제)를 참조하십시오.**

    * `open_eof_check/open_length_check/open_http_protocol`와 같은 하위 수준의 설정을 사용하여 데이터 패킷의 완정성을 보장할 수 있음
    * 하위 수준의 프로토콜 처리를 사용하지 않고, PHP 코드 내에서 데이터를 자체적으로 분석하고 데이터 패킷을 병합/분할해야 함.

    예를 들어: 코드에 `$buffer = array()`를 추가하고, `$fd`를 `key`로 사용하여 컨텍스트 데이터를 저장할 수 있음. 데이터를 수신할 때마다 문자열을 연결하고, `$buffer[$fd] .= $data`를 사용하여 `$buffer[$fd]` 문자열이 완전한 데이터 패킷인지 확인해야 함.

    기본적으로 동일한 `fd`는 동일한 `Worker`에 할당됩니다. 따라서 데이터를 연결할 수 있습니다. `dispatch_mode`를 `3`으로 설정했을 때, 요청 데이터는 선점적으로 처리되므로 동일한 `fd`에서 수신된 데이터가 다른 프로세스로 분배될 수 있어서 위의 데이터 패킷 연결 방법을 사용할 수 없습니다.

  * **다중 포트 수신에 대한 참조는 [여기를](/server/port) 참조하십시오.**

    주 서버가 프로토콜을 설정한 경우, 추가로 수신하는 포트는 기본적으로 주 서버의 설정을 상속받습니다. 포트의 프로토콜을 다시 설정하려면 `set` 메소드를 명시적으로 호출해야 합니다.

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9501);
    $port2 = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $port2->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        echo "[#".$server->worker_id."]\tClient[$fd]: $data\n";
    });
    ```

    여기서 `on` 메소드를 호출하여 [onReceive](/server/events?id=onreceive) 콜백 함수를 등록하였지만, 주 서버의 프로토콜을 재정의하지 않았기 때문에 새로운 리스닝된 `9502` 포트는 여전히 `HTTP` 프로토콜을 사용합니다. `telnet` 클라이언트를 사용하여 `9502` 포트에 문자열을 보낼 때 서버에서는 [onReceive](/server/events?id=onreceive)가 발생하지 않습니다.

  * **주의**

    !> 자동 프로토콜 옵션을 활성화하지 않은 경우, [onReceive](/server/events?id=onreceive)로 수신되는 데이터의 최대 크기는 `64K`입니다.  
    자동 프로토콜 처리 옵션을 활성화하면, [onReceive](/server/events?id=onreceive)는 전체 데이터 패킷을 수신하며 최대 크기는 [package_max_length](/server/setting?id=package_max_length)를 초과하지 않습니다.  
    이진 형식을 지원하며, `$data`는 이진 데이터가 될 수 있습니다.
## onPacket

?> **UDP 데이터 패킷을 수신할 때 이 함수가 콜백되며, `worker` 프로세스에서 발생합니다.**

```php
function onPacket(Swoole\Server $server, string $data, array $clientInfo);
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본값**：없음
      * **기타 값**：없음

    * **`string $data`**
      * **기능**：받은 데이터 내용 (텍스트 또는 이진 콘텐츠 가능)
      * **기본값**：없음
      * **기타 값**：없음

    * **`array $clientInfo`**
      * **기능**：클라이언트 정보에는 `address/port/server_socket`과 같은 여러 클라이언트 정보 데이터가 포함됩니다. [UDP 서버 참조](/start/start_udp_server)
      * **기본값**：없음
      * **기타 값**：없음

  * **주의**

    !> 서버가 동시에 `TCP/UDP` 포트를 수신할 때 `TCP` 프로토콜 데이터가 도착하면 [onReceive](/server/events?id=onreceive)가 콜백되고, `UDP` 데이터 패킷이 도착하면 `onPacket`이 콜백됩니다. 서버에서 설정한 `EOF` 또는 `Length` 등의 자동 프로토콜 처리([TCP 데이터 패킷 경계 문제 참조](/learn?id=tcp数据包边界问题))는 `UDP` 포트에는 적용되지 않습니다. 왜냐하면 `UDP` 패킷 자체에는 메시지 경계가 존재하며, 추가 프로토콜 처리가 필요하지 않기 때문입니다.
## onClose

?> **TCP 클라이언트가 연결을 닫은 후, Worker 프로세스에서 이 함수가 콜백됩니다.**

```php
function onClose(Swoole\Server $server, int $fd, int $reactorId);
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본 값**：없음
      * **기타 값**：없음

    * **`int $fd`**
      * **기능**：연결의 파일 디스크립터
      * **기본 값**：없음
      * **기타 값**：없음

    * **`int $reactorId`**
      * **기능**：어느 `reactor` 스레드에서 왔는지, 서버가 `close`를 사용하여 닫은 경우 음수
      * **기본 값**：없음
      * **기타 값**：없음

  * **팁**

    * **액티브 클로즈**

      * 서버가 연결을 액티브하게 닫을 때, 하위 단계에서 이 매개변수를 `-1`로 설정하고, `$reactorId < 0`로 확인하여 닫힘이 서버 측에서 인지된 것인지 클라이언트 측에서 발생한 것인지 구별할 수 있습니다.
      * `close` 메서드를 PHP 코드에서 액티브하게 호출하는 경우에만 액티브 클로즈로 간주됩니다.

    * **하트비트 체크**

      * [하트비트 체크](/server/setting?id=heartbeat_check_interval)는 하트비트 체크 스레드에서 종료되며, 닫힐 때 [onClose](/server/events?id=onclose)의 `$reactorId` 매개변수는 `-1`이 아닙니다.

  * **주의**

    !> - [onClose](/server/events?id=onclose) 콜백 함수에서 치명적인 오류가 발생하면 연결이 누수될 수 있습니다. `netstat` 명령을 통해 많은 `CLOSE_WAIT` 상태의 TCP 연결을 볼 수 있습니다. [Swoole 비디오 강의](https://course.swoole-cloud.com/course-video/4)를 참조하십시오.  
    - 클라이언트가 `close`를 호출하든 서버가 `$server->close()`를 호출하든 연결을 닫으면 이 이벤트가 트리거됩니다. 즉, 연결이 닫히면 항상 이 함수가 콜백됩니다.  
    - [onClose](/server/events?id=onclose)에서는 여전히 [getClientInfo](/server/methods?id=getClientInfo) 메서드를 호출하여 연결 정보를 얻을 수 있으며, [onClose](/server/events?id=onclose) 콜백 함수가 완료된 후에 `close`를 사용하여 TCP 연결을 닫습니다.  
    - 여기서 [onClose](/server/events?id=onclose)를 콜백할 때는 이미 클라이언트 연결이 닫혔기 때문에 `$server->close($fd)`를 실행할 필요가 없습니다. 이 코드는 `PHP` 오류 경고를 발생시킵니다.
## onTask

?> **`task` 프로세스 내에서 호출됩니다. `worker` 프로세스는 [task](/server/methods?id=task) 함수를 사용하여 `task_worker` 프로세스로 새 작업을 전달할 수 있습니다. 현재의 [Task 프로세스](/learn?id=taskworker프로세스)는 [onTask](/server/events?id=ontask) 콜백 함수를 호출할 때 프로세스 상태를 사용 중으로 전환하며, 이때는 새 Task를 더 이상 수신하지 않습니다. [onTask](/server/events?id=ontask) 함수가 반환될 때 프로세스 상태를 다시 비활성화로 전환하고 새 `Task`를 계속해서 수신합니다.**

```php
function onTask(Swoole\Server $server, int $task_id, int $src_worker_id, mixed $data);
```

  * **Parameter** 

    * **`Swoole\Server $server`**
      * **Description**：Swoole\Server 객체
      * **Default value**：없음
      * **Other values**：없음

    * **`int $task_id`**
      * **Description**：수행중인 작업을 하는 `task` 프로세스 `id`【`$task_id`와 `$src_worker_id`를 조합해야 전역적으로 고유하며, 다른 `worker` 프로세스에서 전달하는 작업 `ID`가 동일할 수 있음】
      * **Default value**：없음
      * **Other values**：없음

    * **`int $src_worker_id`**
      * **Description**：작업을 전달한 `worker` 프로세스 `id`【`$task_id`와 `$src_worker_id`를 조합해야 전역적으로 고유하며, 다른 `worker` 프로세스에서 전달하는 작업 `ID`가 동일할 수 있음】
      * **Default value**：없음
      * **Other values**：없음

    * **`mixed $data`**
      * **Description**：작업 데이터 내용
      * **Default value**：없음
      * **Other values**：없음

  * **Tip**

    * **v4.2.12부터 [task_enable_coroutine](/server/setting?id=task_enable_coroutine)를 활성화하면 콜백 함수 원형은**

      ```php
      $server->on('Task', function (Swoole\Server $server, Swoole\Server\Task $task) {
          var_dump($task);
          $task->finish([123, 'hello']); //작업을 완료하고 데이터 반환
      });
      ```

    * **작업 결과를 `worker` 프로세스에 반환**

      * **[onTask](/server/events?id=ontask) 함수 안에서 `return` 문자열을 사용하면 작업 내용이 `worker` 프로세스에 반환됩니다. `worker` 프로세스에서는 [onFinish](/server/events?id=onfinish) 함수가 트리거됩니다. 물론 `Swoole\Server->finish()`을 사용하여 다시 `return` 없이 [onFinish](/server/events?id=onfinish) 함수를 트리거할 수 있습니다.**

      * `return`하는 변수는 `null`이 아닌 임의의 `PHP` 변수일 수 있습니다

  * **주의**

    !> [onTask](/server/events?id=ontask) 함수가 실행 중致명적 오류로 종료되거나 외부 프로세스에 의해 강제로 `kill`되면 현재 작업은 삭제되지만 대기 중인 다른 `Task`에는 영향을 주지 않습니다
## onFinish

?> **이 콜백 함수는 `worker` 프로세스에서 호출됩니다. `worker` 프로세스가 `task` 프로세스에서 작업을 완료하면, [task 프로세스](/learn?id=taskworker프로세스)는 `Swoole\Server->finish()` 메서드를 사용하여 작업 처리 결과를 `worker` 프로세스에 전송합니다.**

```php
function onFinish(Swoole\Server $server, int $task_id, mixed $data)
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $task_id`**
      * **기능**：작업을 실행한 `task` 프로세스의 `id`
      * **기본값**：없음
      * **다른 값**：없음

    * **`mixed $data`**
      * **기능**：작업 처리 결과 내용
      * **기본값**：없음
      * **다른 값**：없음

  * **주의사항**

    !> - [task 프로세스](/learn?id=taskworker프로세스)의 [onTask](/server/events?id=ontask) 이벤트에서 `finish` 메서드를 호출하거나 결과를 `return`하지 않으면, `worker` 프로세스는 [onFinish](/server/events?id=onfinish)를 트리거하지 않습니다.
    -[onFinish](/server/events?id=onfinish) 로직을 수행하는 `worker` 프로세스는 작업을 보낸 `task` 작업자의 `worker` 프로세스와 동일한 프로세스입니다.
## onPipeMessage

?> **`$server->sendMessage()`로 보낸 [unixSocket](/learn?id=IPC이란 무엇인가요) 메시지를 작업 프로세스가 수신하면 `onPipeMessage` 이벤트가 트리거됩니다. `worker/task` 프로세스는 모두 `onPipeMessage` 이벤트를 트리거할 수 있습니다.**

```php
function onPipeMessage(Swoole\Server $server, int $src_worker_id, mixed $message);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**：Swoole\Server 객체
      * **Default**：없음
      * **Others**：없음

    * **`int $src_worker_id`**
      * **Description**：메시지가 어느 `Worker` 프로세스에서 왔는지
      * **Default**：없음
      * **Others**：없음

    * **`mixed $message`**
      * **Description**：메시지 내용, 임의의 PHP 유형이 될 수 있음
      * **Default**：없음
      * **Others**：없음
## onWorkerError

?> **`Worker/Task` 프로세스에서 예외가 발생하면 `Manager` 프로세스 내에서이 함수를 콜백합니다.**

!> 이 함수는 경보 및 모니터링에 사용되며, Worker 프로세스가 예기치 않게 종료된 경우에는 심각한 오류나 프로세스 Core Dump에 직면한 경우가 많습니다. 로그를 기록하거나 경고 메시지를 보내어 개발자가 적절한 조치를 취할 수 있도록 알릴 수 있습니다.

```php
function onWorkerError(Swoole\Server $server, int $worker_id, int $worker_pid, int $exit_code, int $signal);
```

  * **매개변수** 

    * **`Swoole\Server $server`**
      * **기능**: Swoole\Server 객체
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`int $worker_id`**
      * **기능**: 예외가 발생한 `worker` 프로세스의 `id`
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`int $worker_pid`**
      * **기능**: 예외가 발생한 `worker` 프로세스의 `pid`
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`int $exit_code`**
      * **기능**: 종료 상태 코드, 범위는 `0～255`
      * **기본값**: 없음
      * **기타 값**: 없음

    * **`int $signal`**
      * **기능**: 프로세스 종료 신호
      * **기본값**: 없음
      * **기타 값**: 없음

  * **일반 오류**

    * `signal = 11` : `Worker` 프로세스에 `segment fault` 세그먼트 오류가 발생했음을 나타냅니다. 이는 하위 레벨 `BUG`를 일으킬 수 있으므로 `core dump` 정보 및 `valgrind` 메모리 검사 로그를 수집하고, [Swoole 개발팀에이 문제를 피드백](/other/issue)해야합니다.
    * `exit_code = 255`: Worker 프로세스에 `Fatal Error`致명 오류가 발생했음을 나타냅니다. PHP 오류 로그를 확인하고 문제가 있는 PHP 코드를 해결하십시오.
    * `signal = 9`: `Worker`가 시스템에 의해 강제로 `Kill`되었음을 나타냅니다. 의도적인 `kill -9` 작업이 있는지 확인하고, `OOM(메모리 부족)`이 `dmesg` 정보에 있는지 확인하십시오.
    * 만약 `OOM`이 있는 경우에는 너무 많은 메모리를 할당했습니다. 1. `Server`의 `setting` 구성을 확인하여 [socket_buffer_size](/server/setting?id=socket_buffer_size)가 너무 큰지 확인하십시오. 2. 매우 큰 [Swoole\Table](/memory/table) 메모리 모듈을 만들었는지 확인하십시오.
## onManagerStart

?> **이 이벤트는 관리자 프로세스가 시작될 때 트리거됩니다**

```php
function onManagerStart(Swoole\Server $server);
```

  * **팁**

    * 이 콜백 함수에서는 관리자 프로세스의 이름을 변경할 수 있습니다.
    * `4.2.12` 이전 버전에서는 `manager` 프로세스에서 타이머를 추가할 수 없으며 task 작업을 보낼 수 없으며 코루틴을 사용할 수 없습니다.
    * `4.2.12` 또는 그 이상의 버전에서는 `manager` 프로세스가 신호를 기반으로 한 동기식 타이머를 사용할 수 있습니다.
    * `manager` 프로세스에서 [sendMessage](/server/methods?id=sendMessage) 인터페이스를 호출하여 다른 작업 프로세스로 메시지를 전송할 수 있습니다.

    * **시작 순서**

      * `Task` 및 `Worker` 프로세스가 생성됨
      * `Master` 프로세스 상태가 알려지지 않음, 왜냐하면 `Manager`와 `Master`는 병렬이기 때문에 `onManagerStart` 콜백이 호출될 때 `Master` 프로세스가 준비된 것을 보장할 수 없음

    * **BASE 모드**

      * [SWOOLE_BASE](/learn?id=swoole_base) 모드에서, `worker_num`、`max_request`、`task_worker_num` 매개변수가 설정되어 있는 경우 하위 수준에서는 작업 프로세스를 관리하기 위해 `manager` 프로세스를 만듭니다. 따라서 `onManagerStart`와 `onManagerStop` 이벤트 콜백이 트리거됩니다.
## onManagerStop

?> **매니저 프로세스가 종료될 때 실행됩니다**

```php
function onManagerStop(Swoole\Server $server);
```

 * **팁**

  * `onManagerStop`이 실행되면`Task` 및 `Worker` 프로세스가 종료되었고, `Manager` 프로세스에 의해 회수되었다는 것을 의미합니다.
## onBeforeReload

?> **Worker process`Reload`이전에이 이벤트가 발생하며 Manager 프로세스에서 콜백됩니다.**

```php
function onBeforeReload(Swoole\Server $server);
```

  * **매개변수**

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본값**：없음
      * **다른 값**：없음
## onAfterReload

?> **Worker 프로세스 `Reload` 후에이 이벤트가 발생하며 Manager 프로세스에서 콜백됩니다.**

```php
function onAfterReload(Swoole\Server $server);
```

  * **파라미터**

    * **`Swoole\Server $server`**
      * **기능**：Swoole\Server 객체
      * **기본값**：없음
      * **다른 값**：없음
## 이벤트 실행 순서

* 모든 이벤트 콜백은 `$server->start` 이후에 발생합니다.
* 서버가 종료될 때 마지막 이벤트는 `onShutdown`입니다.
* 서버가 성공적으로 시작된 후, `onStart/onManagerStart/onWorkerStart` 이 서로 다른 프로세스에서 동시에 실행됩니다.
* `onReceive/onConnect/onClose`는 `Worker` 프로세스에서 트리거됩니다.
* `Worker/Task` 프로세스가 시작/종료될 때 각각 `onWorkerStart/onWorkerStop`이 한 번씩 호출됩니다.
* [onTask](/server/events?id=ontask) 이벤트는 [작업자(worker) 프로세스](/learn?id=taskworker进程)에서만 발생합니다.
* [onFinish](/server/events?id=onfinish) 이벤트는 `worker` 프로세스에서만 발생합니다.
* `onStart/onManagerStart/onWorkerStart` 이 3개 이벤트의 실행 순서는 결정되어 있지 않습니다.
## 객체 지향 스타일

[event_object](/server/setting?id=event_object)을 활성화하면 다음 이벤트 콜백이 객체 스타일을 사용합니다.
#### [Swoole\Server\Event](/server/event_class)

* [onConnect](/server/events?id=onconnect)
* [onReceive](/server/events?id=onreceive)
* [onClose](/server/events?id=onclose)

```php
$server->on('Connect', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});

$server->on('Receive', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});

$server->on('Close', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});
```
```php
$server->on('Packet', function (Swoole\Server $serv, Swoole\Server\Packet $object) {
    var_dump($object);
});
```  
#### [Swoole\Server\PipeMessage](/server/pipemessage_class)

* [onPipeMessage](/server/events?id=onpipemessage)

```php
$server->on('PipeMessage', function (Swoole\Server $serv, Swoole\Server\PipeMessage $msg) {
    var_dump($msg);
    $object = $msg->data;
    $serv->sendto($object->address, $object->port, $object->data, $object->server_socket);
});
```
#### [Swoole\Server\StatusInfo](/server/statusinfo_class)

* [onWorkerError](/server/events?id=onworkererror)

```php
$serv->on('WorkerError', function (Swoole\Server $serv, Swoole\Server\StatusInfo $info) {
    var_dump($info);
});
```  
#### [Swoole\Server\Task](/server/task_class)

* [onTask](/server/events?id=ontask)

```php
$server->on('Task', function (Swoole\Server $serv, Swoole\Server\Task $task) {
    var_dump($task);
});
```
#### [Swoole\Server\TaskResult](/server/taskresult_class)

* [onFinish](/server/events?id=onfinish)

```php
$server->on('Finish', function (Swoole\Server $serv, Swoole\Server\TaskResult $result) {
    var_dump($result);
});
```
