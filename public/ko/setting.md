# 구성

[Swoole\Server->set()](/server/methods?id=set) 함수는 `Server`가 실행될 때 사용되는 여러 매개변수를 설정하는 데 사용됩니다. 이 섹션의 모든 하위 페이지는 구성 배열의 요소입니다.

!> [v4.5.5](/version/log?id=v455) 버전부터, 기본적으로 설정된 구성 항목의 유효성을 검사하며, `Swoole`이 제공하지 않는 구성 항목을 설정하면 경고가 발생합니다.

```shell
PHP Warning:  unsupported option [foo] in @swoole-src/library/core/Server/Helper.php 
```
### debug_mode

?> 로깅 모드를 `debug` 디버그 모드로 설정하고, 컴파일 시 `--enable-debug`로 활성화해야만 작동합니다.

```php
$server->set([
  'debug_mode' => true
])
```  
### trace_flags

?> 추적 로그의 태그를 설정하여 일부 추적 로그만 인쇄합니다. `trace_flags`는 여러 추적 항목을 설정하는 데 `|` 비트 연산자를 사용할 수 있습니다. 컴파일 시 `--enable-trace-log`를 활성화해야만 작동합니다.

다음은 사용 가능한 추적 항목입니다. `SWOOLE_TRACE_ALL`을 사용하여 모든 항목을 추적할 수 있습니다:

* `SWOOLE_TRACE_SERVER`
* `SWOOLE_TRACE_CLIENT`
* `SWOOLE_TRACE_BUFFER`
* `SWOOLE_TRACE_CONN`
* `SWOOLE_TRACE_EVENT`
* `SWOOLE_TRACE_WORKER`
* `SWOOLE_TRACE_REACTOR`
* `SWOOLE_TRACE_PHP`
* `SWOOLE_TRACE_HTTP2`
* `SWOOLE_TRACE_EOF_PROTOCOL`
* `SWOOLE_TRACE_LENGTH_PROTOCOL`
* `SWOOLE_TRACE_CLOSE`
* `SWOOLE_TRACE_HTTP_CLIENT`
* `SWOOLE_TRACE_COROUTINE`
* `SWOOLE_TRACE_REDIS_CLIENT`
* `SWOOLE_TRACE_MYSQL_CLIENT`
* `SWOOLE_TRACE_AIO`
* `SWOOLE_TRACE_ALL`
### log_file

?> **`Swoole` 오류 로그 파일 지정**

?> `Swoole`이 실행될 때 발생한 예외 정보는 이 파일에 기록되며, 기본적으로 화면에 출력됩니다.  
데몬 모드를 활성화한 경우`(daemonize => true)`, 표준 출력은 `log_file`로 리다이렉트됩니다. PHP 코드에서 `echo/var_dump/print`로 화면에 출력한 내용은 `log_file` 파일에 기록됩니다.

  * **팁**

    * `log_file`에 기록된 로그는 런타임 오류 기록용이며 장기 저장이 필요하지 않습니다.

    * **로그 마크**

      ?> 로그 정보에는 프로세스 ID 앞에 특정 문자가 추가되어 있으며, 로그가 생성된 스레드/프로세스 유형을 나타냅니다.

        * `#` 마스터 프로세스
        * `$` 매니저 프로세스
        * `*` 워커 프로세스
        * `^` 태스크 프로세스

    * **로그 파일 다시 열기**

      ?> 서버 프로그램이 실행되는 동안 로그 파일이 `mv`로 이동되거나 `unlink`로 삭제된 경우, 로그 정보가 올바르게 기록되지 않습니다. 이때 `Server`로 `SIGRTMIN` 신호를 보내어 로그 파일을 다시 열 수 있습니다.

      * 리눅스 플랫폼만 지원됨
      * [UserProcess](/server/methods?id=addProcess) 프로세스는 지원되지 않음

  * **주의**

    !> `log_file`은 파일을 자동으로 분할하지 않으므로 주기적으로 이 파일을 정리해야 합니다. `log_file` 출력을 관찰하여 서버의 다양한 예외 정보와 경고를 얻을 수 있습니다.
### 로그 레벨

?> **`Server` 오류 로그를 출력하는 레벨을 설정합니다. 범위는 `0-6`입니다. `log_level`로 설정한 값보다 낮은 로그 정보는 발생하지 않습니다.**【기본값: `SWOOLE_LOG_INFO`】

대응 레벨 상수에 대한 자세한 내용은 [로그 레벨](/consts?id=로그-레벨)을 참조하십시오.

  * **주의**

    !> `SWOOLE_LOG_DEBUG` 및 `SWOOLE_LOG_TRACE`는 [--enable-debug-log](/environment?id=디버그-매개변수) 및 [--enable-trace-log](/environment?id=디버그-매개변수)로 컴파일된 버전에서만 사용할 수 있습니다.  
    `daemonize` 데몬화를 활성화할 경우에는 백그라운드에서 모든 화면 출력 내용을 [log_file](/server/setting?id=log_file)에 기록하며, 이 부분은 `log_level`의 제어를 받지 않습니다.
### log_date_format

?> **`Server`의 로그 시간 형식 설정**, 형식은 [strftime](https://www.php.net/manual/zh/function.strftime.php)의`format`을 참조하세요.

```php
$server->set([
    'log_date_format' => '%Y-%m-%d %H:%M:%S',
]);
```
### log_date_with_microseconds

?> **`Server` 로그의 마이크로초 정밀도를 설정합니다.**【기본값: `false`】
### 로그 회전

?> **`Server` 로그 분할 설정**【기본값: `SWOOLE_LOG_ROTATION_SINGLE`】

| 상수                             | 설명   | 버전 정보 |
| -------------------------------- | ------ | -------- |
| SWOOLE_LOG_ROTATION_SINGLE       | 비활성화 | -        |
| SWOOLE_LOG_ROTATION_MONTHLY      | 매월    | v4.5.8   |
| SWOOLE_LOG_ROTATION_DAILY        | 매일    | v4.5.2   |
| SWOOLE_LOG_ROTATION_HOURLY       | 매시간  | v4.5.8   |
| SWOOLE_LOG_ROTATION_EVERY_MINUTE | 매분    | v4.5.8   |
### display_errors

?> `Swoole` 오류 메시지를 표시하거나 숨깁니다.

```php
$server->set([
  'display_errors' => true
])
```
### dns_server

?> `dns` 쿼리의 `ip` 주소를 설정하십시오.
### socket_dns_timeout

?> 도메인 해석 시간 초과, 서버에서 코루틴 클라이언트를 활성화하는 경우이 매개 변수는 클라이언트의 도메인 해석 시간 초과를 제어 할 수 있습니다. 단위 : 초.
### socket_connect_timeout

?> 클라이언트 연결 타임아웃, 서버에서 코루틴 클라이언트를 활성화할 경우 이 매개변수는 클라이언트 연결 타임아웃을 제어할 수 있으며, 단위는 초입니다.
### socket_write_timeout / socket_send_timeout

?> 클라이언트 쓰기 타임아웃은 서버에서 코루틴 클라이언트를 활성화한 경우 클라이언트의 쓰기 타임아웃을 제어할 수 있는 매개변수이며 단위는 초입니다.   
이 구성은 `코루틴`을 적용한 이후의 `shell_exec` 또는 [Swoole\Coroutine\System::exec()](/coroutine/system?id=exec)의 실행 타임아웃 시간을 제어하는 데에도 사용될 수 있습니다.
### 소켓 읽기 시간 초과 / 소켓 수신 시간 초과

?> 클라이언트 읽기 시간 초과는 서버에서 코루틴 클라이언트를 사용하는 경우 클라이언트의 읽기 시간을 제어할 수 있는 매개변수이며, 단위는 초입니다.
### max_coroutine / max_coro_num :id=max_coroutine

?> **현재 워커 프로세스의 최대 코루틴 수를 설정합니다.**【기본값: `100000`, `Swoole 버전이`v4.4.0-beta`보다 낮을 때 기본값은`3000`】

?> `max_coroutine`을 초과하면 하부에서 새로운 코루틴을 생성할 수 없으며, Swoole 서버는 `exceed max number of coroutine` 오류를 발생시키고, `TCP Server`는 연결을 직접 닫습니다. `Http Server`는 Http의 503 상태 코드를 반환합니다.

?> `Server` 프로그램에서 실제로 생성 가능한 최대 코루틴 수는 `worker_num * max_coroutine`와 같습니다. task 프로세스와 UserProcess 프로세스의 코루틴 수는 개별적으로 계산됩니다.

```php
$server->set(array(
    'max_coroutine' => 3000,
));
```
### enable_deadlock_check

?> 코루틴 데드락 검사를 활성화합니다.

```php
$server->set([
  'enable_deadlock_check' => true
]);
```
### hook_flags

?> **`일괄적으로 코루틴으로 만들기` 훅의 함수 범위를 설정합니다.**【기본값: 훅되지 않음】

!> Swoole 버전은 `v4.5+` 또는 [4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x)에서 사용할 수 있습니다. 자세한 내용은 [일괄적으로 코루틴으로 만들기](/runtime)를 참조하십시오.

```php
$server->set([
    'hook_flags' => SWOOLE_HOOK_SLEEP,
]);
```
다음과 같은 항목을 지원하며, `SWOOLE_HOOK_ALL`을 사용하여 모두 코루틴으로 만들 수 있습니다:

* `SWOOLE_HOOK_TCP`
* `SWOOLE_HOOK_UNIX`
* `SWOOLE_HOOK_UDP`
* `SWOOLE_HOOK_UDG`
* `SWOOLE_HOOK_SSL`
* `SWOOLE_HOOK_TLS`
* `SWOOLE_HOOK_SLEEP`
* `SWOOLE_HOOK_FILE`
* `SWOOLE_HOOK_STREAM_FUNCTION`
* `SWOOLE_HOOK_BLOCKING_FUNCTION`
* `SWOOLE_HOOK_PROC`
* `SWOOLE_HOOK_CURL`
* `SWOOLE_HOOK_NATIVE_CURL`
* `SWOOLE_HOOK_SOCKETS`
* `SWOOLE_HOOK_STDIO`
* `SWOOLE_HOOK_PDO_PGSQL`
* `SWOOLE_HOOK_PDO_ODBC`
* `SWOOLE_HOOK_PDO_ORACLE`
* `SWOOLE_HOOK_PDO_SQLITE`
* `SWOOLE_HOOK_ALL`
### enable_preemptive_scheduler

?> 코루틴 선점 스케줄러를 활성화하여, 한 코루틴이 실행 시간이 길어 다른 코루틴이 굶어죽는 것을 방지하며, 코루틴 최대 실행 시간은 `10ms` 입니다.

```php
$server->set([
  'enable_preemptive_scheduler' => true
]);
```
### c_stack_size / stack_size

?> 각 코루틴의 초기 C 스택 메모리 크기를 설정합니다. 기본값은 2M입니다.
### aio_core_worker_num

?> `AIO`의 최소 작업 스레드 수를 설정하며, 기본값은 `cpu` 코어 수입니다.
### aio_worker_num 

?> `AIO` 최대 작업 스레드 수를 설정합니다. 기본값은 `cpu` 코어 수 * 8 입니다.
### aio_max_wait_time

?> 작업 스레드가 작업을 기다리는 최대 시간(초)입니다.
### aio_max_idle_time

?> 작업 스레드의 최대 유휴 시간(초 단위)입니다.
### reactor_num

?> **[Reactor](/learn?id=reactor线程) 스레드를 설정합니다.**【기본값: `CPU` 코어 수】

?> 이 매개변수를 사용하여 주 프로세스 내 이벤트 처리 스레드의 수를 조절하여 다중 코어를 최대한 활용합니다. 기본적으로 `CPU` 코어 수와 동일한 수의 스레드를 시작합니다.  
`Reactor` 스레드는 다중 코어를 활용할 수 있으며, 예를 들어, 기기에 `128`개의 코어가 있는 경우 백그라운드에서 `128`개의 스레드가 시작됩니다.  
각 스레드는 [EventLoop](/learn?id=什么是eventloop)를 유지할 수 있습니다. 스레드 간에는 락이 없으며, 명령은 `128`개의 코어 `CPU`에서 병렬로 실행될 수 있습니다.  
운영 체제 스케줄링에는 일부 성능 손실이 있는 것을 감안하여, CPU 코어 수*2로 설정하여 각 코어를 최대로 활용합니다.

  * **팁**

    * `reactor_num`을 `CPU` 코어 수의 `1-4`배로 설정하는 것이 좋습니다.
    * `reactor_num`은 [swoole_cpu_num()](/functions?id=swoole_cpu_num) * 4를 초과해서 설정할 수 없습니다.

  * **주의**

  !> -`reactor_num`은 `worker_num`보다 작거나 같아야 합니다.  
-만약 설정한 `reactor_num`이 `worker_num`보다 크다면, 자동으로 조정되어 `reactor_num`이 `worker_num`과 같아집니다.  
-`8`개 이상의 코어를 가진 기기에서 `reactor_num`은 기본적으로 `8`로 설정됩니다.
### worker_num

?> **`Worker` 프로세스를 시작하는 데 사용할 프로세스 수를 설정합니다.**【기본값: `CPU` 코어 수】

?> 만약 `1`개의 요청이 `100ms`가 소요되고, `1000QPS`의 처리 능력을 제공하려면, 적어도 `100`개 이상의 프로세스를 설정해야 합니다.  
그러나 시작하는 프로세스가 많을수록 메모리 사용량도 크게 증가하고, 프로세스 간 전환 비용도 높아집니다. 따라서 적당히만 설정하십시오. 과도하게 설정하지 마십시오.

  * **팁**

    * 비즈니스 코드가 완전히 [비동기IO](/learn?id=동기io와-비동기io)인 경우, 이 값을 `CPU` 코어 수의 `1-4`배로 설정하는 것이 가장 합리적입니다.
    * 비즈니스 코드가 [동기IO](/learn?id=동기io와-비동기io)인 경우, 요청 응답 시간과 시스템 부하에 따라 조정해야 합니다. 예를 들어 `100-500`입니다.
    * 기본 설정 값은 [swoole_cpu_num()](/functions?id=swoole_cpu_num)으로 설정되며, 최대 값은 [swoole_cpu_num()](/functions?id=swoole_cpu_num) * 1000을 초과할 수 없습니다.
    * 각 프로세스가 `40M`의 메모리를 사용한다고 가정하면, `100`개의 프로세스는 `4G`의 메모리를 사용하게 됩니다. 프로세스의 메모리 사용량을 올바르게 확인하는 방법은 [Swoole 공식 비디오 강좌](https://course.swoole-cloud.com/course-video/85)를 참조하십시오.
### max_request

?> **`worker` 프로세스의 최대 요청 수를 설정합니다.**【기본값: `0` 즉, 프로세스가 종료되지 않습니다】

?> 이 수치를 초과한 작업을 처리한 후 `worker` 프로세스는 자동으로 종료되며, 프로세스가 종료되면 모든 메모리 및 리소스가 해제됩니다.

!> 이 매개변수의 기본 역할은 프로그램 인코딩 문제로 인한 PHP 프로세스 메모리 누수를 해결하는 것입니다. PHP 응용 프로그램은 천천히 메모리 누출을 발생시키지만 구체적인 원인을 찾거나 해결할 수 없는 경우 `max_request`를 설정하여 일시적으로 해결할 수 있습니다. 메모리 누수 코드를 찾아 수정해야 하며, 이 방안 대신 [Swoole Tracker](https://course.swoole-cloud.com/course-video/92)를 사용하여 누수 코드를 발견할 수 있습니다.

  * **팁**

    * `max_request`에 도달한 경우 즉시 프로세스를 종료하지 않을 수 있습니다. [max_wait_time](/server/setting?id=max_wait_time)을 참조하십시오.
    * [SWOOLE_BASE](/learn?id=swoole_base)에서 `max_request`에 도달하면 프로세스를 다시 시작하면 클라이언트 연결이 끊어집니다.

  !> `worker` 프로세스 내에서 치명적인 오류가 발생하거나 수동으로 `exit`를 실행할 때 프로세스가 자동으로 종료됩니다. `master` 프로세스는 새 `worker` 프로세스를 시작하여 요청을 계속 처리합니다.
### max_conn / max_connection

?> **서버 프로그램에서 허용되는 최대 연결 수.**【기본값: `ulimit -n`】

?> 예를 들어 `max_connection => 10000`, 이 매개변수는 `Server`가 최대로 유지할 수 있는 `TCP` 연결의 수를 설정합니다. 이 수를 초과하면 새로운 연결이 거부됩니다.

  * **팁**

    * **기본 설정**

      * 응용 프로그램 계층은 `max_connection`을 설정하지 않으면 하위 수준에서 `ulimit -n` 값이 기본 설정으로 사용됩니다.
      * `4.2.9` 이상 버전에서는 하위 수준에서 `ulimit -n`이 `100000`을 초과하면 기본 설정을 `100000`으로 설정합니다. 이유는 일부 시스템이 `ulimit -n`을 `1 백만`으로 설정하여 많은 메모리를 할당하고 시작 오류가 발생하기 때문입니다.

    * **최대 상한선**

      * `max_connection`을 `1M`을 초과하여 설정하지 마십시오.

    * **최소 설정**
    
      * 이 옵션을 너무 작게 설정하면 하위 수준에서 오류가 발생하고 `ulimit -n`의 값으로 설정됩니다.
      * 최소값은 `(worker_num + task_worker_num) * 2 + 32` 입니다.

    ```shell
    serv->max_connection이 너무 작습니다.
    ```

    * **메모리 사용량**

      * `max_connection` 매개변수를 지나치게 크게 조정하지 마십시오. 기계 메모리 상황에 따라 설정하십시오. `Swoole`은 이 값을 기반으로 `Connection` 정보를 저장하기 위해 큰 메모리 블록을 한 번에 할당하며 한 개의 `TCP` 연결의 `Connection` 정보는 `224`바이트를 차지합니다.

  * **주의**

  !> `max_connection`는 운영 체제의 `ulimit -n` 값을 초과해서는 안됩니다. 그러지 않으면 경고 메시지가 표시되고 `ulimit -n` 값으로 재설정됩니다.

  ```shell
  WARN swServer_start_check: serv->max_conn is exceed the maximum value[100000].

  WARNING set_max_connection: max_connection is exceed the maximum value, it's reset to 10240
  ```
### task_worker_num

?> **Configure the number of [Task workers](/learn?id=taskworker-process).**

?> After configuring this parameter, the `task` function will be enabled. Therefore, the `Server` must register the [onTask](/server/events?id=ontask) and [onFinish](/server/events?id=onfinish) 2 event callback functions. If not registered, the server program will fail to start.

  * **Tips**

    * [Task workers](/learn?id=taskworker-process) are synchronous and blocking.

    * The maximum value should not exceed [swoole_cpu_num()](/functions?id=swoole_cpu_num) * 1000.
    
    * **Calculation Method**
      * If a single `task` processing takes `100ms`, then a process can handle `1/0.1=10` tasks per second.
      * If `2000` tasks are generated per second.
      * `2000/10=200`, you need to set `task_worker_num => 200` to enable `200` Task workers.

  * **Note**

    !> - The `Swoole\Server->task` method cannot be used in [Task workers](/learn?id=taskworker-process).
### task_ipc_mode

?> **작업자 프로세스(Task worker process)와 `Worker` 프로세스 간 통신 방법을 설정합니다.** 【기본값: `1`】

?> 먼저 [Swoole IPC 통신](/learn?id=IPC가 무엇인가요)을 읽어보세요.

모드 | 기능
---|---
1 | `Unix Socket` 통신 사용 【기본 모드】
2 | `sysvmsg` 메시지 대기열 통신 사용
3 | `sysvmsg` 메시지 대기열 통신 사용하며, 경쟁 모드로 설정

  * **팁**

    * **모드`1`**
      * 모드`1`을 사용하는 경우, 방향성 전달이 지원되며, [task](/server/methods?id=task) 및 [taskwait](/server/methods?id=taskwait) 메서드에서 `dst_worker_id`를 사용하여 대상 `Task worker process`를 지정할 수 있습니다.
      * `dst_worker_id`가 `-1`로 설정된 경우, 기본으로 각 [Task worker process](/learn?id=taskworker프로세스)의 상태를 확인하고 현재 공백 상태인 프로세스에 작업을 할당합니다.

    * **모드`2` 및 `3`**
      * 메시지 대기열 모드는 운영 체제가 제공하는 메모리 대기열을 사용하여 데이터를 저장합니다. `mssage_queue_key`를 지정하지 않은 경우, 사설 대기열을 사용하며, `Server` 프로그램이 종료되면 메시지 대기열이 삭제됩니다.
      * 메시지 대기열 `Key`를 지정한 경우, `Server` 프로그램이 종료된 후에도 메시지 대기열에서 데이터가 삭제되지 않으므로 프로세스를 다시 시작하여 데이터를 가져올 수 있습니다.
      * `ipcrm -q` 명령을 사용하여 메시지 대기열 데이터를 수동으로 삭제할 수 있습니다.
      * `모드2`와 `모드3`의 차이점은, `모드2`는 방향성 전달을 지원하며, `$serv->task($data, $task_worker_id)`를 사용하여 어느 [task 프로세스](/learn?id=taskworker프로세스)로 전달할지 지정할 수 있습니다. `모드3`은 완전한 경쟁 모드로, [task 프로세스](/learn?id=taskworker프로세스)가 대기열을 경쟁하므로 방향성 전달을 사용할 수 없으며, `task/taskwait`에서 대상 프로세스 `ID`를 지정할 수 없습니다. 즉, `$task_worker_id`를 지정하더라도 `모드3`에서는 무효입니다.

  * **주의**

    !> - `모드3`을 사용하면 [sendMessage](/server/methods?id=sendMessage) 메서드에 영향을 미쳐 랜덤한 [task 프로세스](/learn?id=taskworker프로세스)가 메시지를 가져갈 수 있습니다.
    - 메시지 대기열 통신 사용 시, `Task worker process`의 처리 능력이 작업 전달 속도보다 낮으면 `Worker` 프로세스가 차단될 수 있습니다.
    - 메시지 대기열 통신 사용 후 task 프로세스는 코루틴을 지원하지 않습니다([task_enable_coroutine](/server/setting?id=task_enable_coroutine) 활성화).
### task_max_request

?> **[작업자 프로세스](/learn?id=taskworker프로세스)의 최대 작업 수를 설정합니다.**【기본값: `0`】

작업자 프로세스의 최대 작업 수를 설정합니다. 이 수 이상의 작업을 처리한 후 작업자 프로세스가 자동으로 종료됩니다. 이 매개변수는 PHP 프로세스의 메모리 오버플로우를 방지하기 위한 것입니다. 프로세스가 자동으로 종료되지 않도록 하려면 0으로 설정하십시오.
### task_tmpdir

?> **task의 데이터 임시 디렉토리를 설정합니다.**【기본값: Linux의 `/tmp` 디렉토리】

?> `Server`에서는 데이터가 `8180`바이트를 초과하면 데이터를 저장하기 위해 임시 파일을 사용합니다. 여기서 `task_tmpdir`은 임시 파일을 저장할 위치를 설정하는 데 사용됩니다.

  * **팁**

    * 기본적으로 `task` 데이터를 저장하기 위해 하위 수준에서는 `/tmp` 디렉토리를 사용합니다. `Linux` 커널 버전이 낮은 경우 `/tmp` 디렉토리가 메모리 파일 시스템이 아니라면 `/dev/shm/`으로 설정할 수 있습니다.
    * `task_tmpdir` 디렉토리가 없는 경우 하위 수준에서 자동으로 생성을 시도합니다.

  * **주의**

    !> - 생성에 실패할 경우, `Server->start`가 실패합니다.
### task_enable_coroutine

?> **`Task` 코루틴 지원 활성화.**【기본값: `false`】, 4.2.12 버전부터 지원됩니다.

?> 활성화된 경우 [onTask](/server/events?id=ontask) 콜백 내에서 자동으로 코루틴 및 [코루틴 스케줄러](/coroutine/scheduler)가 생성되며, `PHP` 코드에서 코루틴 `API`를 직접 사용할 수 있습니다.

  * **예시**

```php
$server->on('Task', function ($serv, Swoole\Server\Task $task) {
    //어느 Worker 프로세스에서 왔는지
    $task->worker_id;
    //작업 번호
    $task->id;
    //작업 유형, taskwait, task, taskCo, taskWaitMulti의 다른 플래그 사용 가능
    $task->flags;
    //작업 데이터
    $task->data;
    //전달 시간, v4.6.0 버전 추가
    $task->dispatch_time;
    //코루틴 API
    co::sleep(0.2);
    //작업 완료, 종료 및 데이터 반환
    $task->finish([123, 'hello']);
});
```

  * **주의**

    !> - `task_enable_coroutine`은 [enable_coroutine](/server/setting?id=enable_coroutine)이 `true` 인 경우에만 사용할 수 있습니다.  
    - `task_enable_coroutine`을 활성화하면 `Task` 작업 프로세스가 코루틴을 지원합니다.  
    - `task_enable_coroutine`을 활성화하지 않으면 동기적인 차단만 지원합니다.
### task_use_object/task_object :id=task_use_object

?> **객체 지향 스타일의 Task 콜백 형식을 사용합니다.**【기본값: `false`】

?> `true`로 설정하면 [onTask](/server/events?id=ontask) 콜백이 객체 모드로 변환됩니다.

  * **예시**

```php
<?php

$server = new Swoole\Server('127.0.0.1', 9501);
$server->set([
    'worker_num'      => 1,
    'task_worker_num' => 3,
    'task_use_object' => true,
//    'task_object' => true, // v4.6.0버전에서 추가된 별칭
]);
$server->on('receive', function (Swoole\Server $server, $fd, $tid, $data) {
    $server->task(['fd' => $fd,]);
});
$server->on('Task', function (Swoole\Server $server, Swoole\Server\Task $task) {
    //여기서 $task는 Swoole\Server\Task 객체입니다.
    $server->send($task->data['fd'], json_encode($server->stats()));
});
$server->start();
```
### dispatch_mode

?> **데이터 패킷 배송 전략입니다.**【기본값: `2`】

모드 값 | 모드 | 기능
---|---|---
1 | 라운드로빈 모드 | 각 `Worker` 프로세스로 순환 배분됩니다
2 | 고정 모드 | 파일 기술자에 따라 `Worker`에 할당됩니다. 이렇게 하면 동일한 연결로부터 수신 된 데이터는 항상 같은 `Worker`에서 처리됩니다
3 | 선접속 모드 | 주 프로세스는 `Worker`의 바쁨 또는 여유 상태에 따라 전달을 선택하며, 유휴 상태인 `Worker`에만 전달됩니다
4 | IP 할당 | 클라이언트 `IP`에 대한 모듈 `hash`를 취하여 특정 `Worker` 프로세스에 할당합니다.<br>동일한 소스 IP의 연결 데이터가 항상 동일한 `Worker` 프로세스로 라우팅되도록 보장합니다. 알고리즘은 `inet_addr_mod(ClientIP, worker_num)`입니다
5 | UID 할당 | 사용자 코드에서 [Server->bind()](/server/methods?id=bind)를 호출하여 연결을 `uid`에 바인딩 해야 합니다. 그런 다음 밑바닥에서는 `UID` 값에 따라 다른 `Worker` 프로세스에 할당됩니다.<br>알고리즘은 `UID % worker_num`이며, 문자열을 `UID`로 사용해야하는 경우 `crc32(UID_STRING)`을 사용할 수 있습니다
7 | 스트림 모드 | 유휴 상태의 `Worker`는 연결을 `accept`하고 [Reactor](/learn?id=reactor선문)의 새 요청을 처리합니다

  * **팁**

    * **사용 권장 사항**
    
      * 무상태 서버는 `1`또는 `3`을 사용할 수 있습니다. 동기 차단 서버는 `3`을 사용하고 비동기 비차단 서버는 `1`을 사용할 수 있습니다
      * 상태를 유지하는 경우 `2`, `4`, `5`를 사용합니다
      
    * **UDP 프로토콜**

      * `dispatch_mode=2/4/5`인 경우 고정 배포이며 하위 수준에서는 클라이언트 `IP`를 사용하여 다른 `Worker` 프로세스로 해싱됩니다
      * `dispatch_mode=1/3`인 경우 서로 다른 `Worker` 프로세스로 무작위로 배포됩니다
      * `inet_addr_mod` 함수

```
    function inet_addr_mod($ip, $worker_num) {
        $ip_parts = explode('.', $ip);
        if (count($ip_parts) != 4) {
            return false;
        }
        $ip_parts = array_reverse($ip_parts);
    
        $ip_long = 0;
        foreach ($ip_parts as $part) {
            $ip_long <<= 8;
            $ip_long |= (int) $part;
        }
    
        return $ip_long % $worker_num;
    }
```

    * **BASE 모드**

      * `dispatch_mode`는 [SWOOLE_BASE](/learn?id=swoole_base) 모드에서는 무효입니다. 왜냐하면 `BASE`에 작업 전송이 존재하지 않으며, 클라이언트에서 데이터를 수신하면 즉시 현재 스레드/프로세스에서 [onReceive](/server/events?id=onreceive) 콜백이 호출되어 `Worker` 프로세스로 전달할 필요가 없습니다.

  * **주의**

    !> - `dispatch_mode=1/3`일 때는 하위 레벨에서 `onConnect/onClose` 이벤트가 마스킹되며, 이는 이러한 2가지 모드에서 `onConnect/onClose/onReceive`의 순서를 보장할 수 없기 때문입니다;  
    - 요청 응답 방식이 아닌 서버 프로그램에서는 모드 `1`또는 `3`을 사용하지 마십시오. 예 : http 서비스는 응답 지향적이므로 `1`또는 `3`을 사용할 수 있지만 TCP 장기 연결 상태의 경우 `1`또는 `3`을 사용할 수 없습니다.
### dispatch_func

?> `dispatch` 함수를 설정하고, `Swoole`의 하위에서는 `6`가지[dispatch_mode](/server/setting?id=dispatch_mode)가 내장되어 있습니다. 필요한 경우에는 `C++` 함수 또는 `PHP` 함수를 작성하여 `dispatch` 로직을 구현할 수 있습니다.

  * **사용 방법**

```php
$server->set(array(
  'dispatch_func' => 'my_dispatch_function',
));
```

  * **팁**

    * `dispatch_func`를 설정하면 내부에서 `dispatch_mode` 구성은 자동으로 무시됩니다.
    * `dispatch_func`에 대응하는 함수가 없으면 하위에서 치명적인 오류가 발생합니다.
    * `8K`를 초과하는 패킷을 `dispatch`해야 하는 경우, `dispatch_func`은 `0-8180` 바이트 내용만 가져올 수 있습니다.

  * **PHP 함수 작성하기**

    ?> `ZendVM`이 멀티 스레드 환경을 지원하지 못하기 때문에, 여러개의 [Reactor](/learn?id=reactor-threads) 스레드를 설정한다 해도 한 번에 하나의 `dispatch_func`만 실행됩니다. 따라서 하위에서 이 PHP 함수를 실행할 때는 잠금 작업이 수행되며, 잠금 경합 문제가 발생할 수 있습니다. `dispatch_func`에는 블로킹 작업을 수행하지 말아야 하며, 그렇지 않으면 `Reactor` 스레드 그룹이 작동을 중단할 수 있습니다.

    ```php
    $server->set(array(
        'dispatch_func' => function ($server, $fd, $type, $data) {
            var_dump($fd, $type, $data);
            return intval($data[0]);
        },
    ));
    ```

    * `$fd`는 고객의 연결 식별자이며, `Server::getClientInfo`를 사용하여 연결 정보를 가져올 수 있습니다.
    * `$type`은 데이터의 유형을 나타내며, `0`은 고객이 보낸 데이터 전송, `4`는 고객 연결 설정, `3`은 고객 연결 닫힘을 나타냅니다.
    * `$data`는 데이터 내용이며, `HTTP`, `EOF`, `Length` 등의 프로토콜 처리 매개변수를 활성화한 경우에는 하위에서 패킷을 조립합니다. 그러나 `dispatch_func` 함수에서는 데이터 패킷의 처음 8K 내용만 전달되므로 완전한 패킷 내용을 얻을 수 없습니다.
    * **반드시** `0 - (server->worker_num - 1)` 사이의 숫자를 반환하여 데이터 패킷의 대상 작업 프로세스 `ID`를 나타냅니다.
    * 0 미만 또는 server->worker_num 이상은 비정상 대상 `ID`이며, `dispatch`된 데이터는 삭제됩니다.

  * **C++ 함수 작성하기**

    **다른 PHP 확장에서는 함수 길이를 Swoole 엔진에 등록하려면 swoole_add_function을 사용합니다.**

    ?> C++ 함수 호출 시에는 하위에서 잠금이 걸리지 않으므로 호출자가 스레드 안전성을 보장해야 합니다.

    ```c++
    int dispatch_function(swServer *serv, swConnection *conn, swEventData *data);

    int dispatch_function(swServer *serv, swConnection *conn, swEventData *data)
    {
        printf("cpp, type=%d, size=%d\n", data->info.type, data->info.len);
        return data->info.len % serv->worker_num;
    }

    int register_dispatch_function(swModule *module)
    {
        swoole_add_function("my_dispatch_function", (void *) dispatch_function);
    }
    ```

    * `dispatch` 함수는 대상 `worker` 프로세스 `ID`를 반환해야 합니다.
    * 반환된 `worker_id`는 `server->worker_num`을 초과해서는 안 되며, 그렇지 않으면 하위에서 세그멘테이션 오류가 발생합니다.
    * 음수를 반환함으로써(`return -1`), 이 데이터 패킷을 폐기함을 나타냅니다.
    * `data`를 통해 이벤트의 유형과 길이를 읽을 수 있습니다.
    * `conn`은 연결 정보이며, `UDP` 데이터 패킷인 경우에는 `NULL`입니다.

  * **주의**

    !> -`dispatch_func`은 [SWOOLE_PROCESS](/learn?id=swoole_process) 모드에서만 유효하며, UDP/TCP/UnixSocket 타입의 서버에서 모두 작동합니다.  
    -반환된 `worker_id`는 `server->worker_num`을 초과해서는 안 되며, 그렇지 않으면 하위에서 세그멘테이션 오류가 발생합니다
### message_queue_key

?> **메시지 큐의 `KEY`를 설정합니다.**【기본값: `ftok($php_script_file, 1)`】

?> `task_ipc_mode`가 2/3 일 때에만 사용됩니다. 설정된 `Key`는 `Task` 작업 큐의 `KEY`로만 사용되며, [Swoole IPC 통신](/learn?id=什么是IPC)을 참조하십시오.

?> `task` 큐는 `server`가 종료된 후에도 파괴되지 않습니다. 프로그램을 다시 시작하면 [task 프로세스](/learn?id=taskworker进程)는 여전히 큐에 있는 작업을 처리합니다. 프로그램을 다시 시작할 때 이전 `Task` 작업을 실행하지 않으려면 이 메시지 큐를 수동으로 삭제할 수 있습니다.

```shell
ipcs -q 
ipcrm -Q [msgkey]
```
### daemonize

?> **데몬화**【기본값: `false`】

?> `daemonize => true`로 설정하면 프로그램이 백그라운드에서 데몬으로 실행됩니다. 오랜 시간 실행되는 서버 프로그램은 이 옵션을 활성화해야 합니다.  
데몬을 사용하지 않으면 ssh 터미널을 종료하면 프로그램이 중지됩니다.

  * **팁**

    * 데몬을 활성화하면 표준 입력과 출력이 `log_file`로 리디렉션됩니다.
    * `log_file`가 설정되지 않은 경우 `/dev/null`로 리디렉션되어 화면에 출력되는 모든 정보가 삭제됩니다.
    * 데몬을 활성화하면 `CWD`(Current Working Directory) 환경 변수가 변경되어 상대 경로의 파일 읽기/쓰기가 실패할 수 있습니다. `PHP` 프로그램에서 절대 경로를 사용해야 합니다.

    * **systemd**

      * `systemd` 또는 `supervisord`를 사용하여 `Swoole` 서비스를 관리할 때 `daemonize => true`를 설정하지 마십시오. 주된 이유는 `systemd`의 메커니즘이 `init`과 다르기 때문입니다. `init` 프로세스의 `PID`는 `1`이며, 프로그램을 `daemonize`하면 터미널에서 분리되어 최종적으로 `init` 프로세스가 호스팅하여 `init`과의 관계가 부모-자식 프로세스 관계로 바뀝니다.
      * 그러나 `systemd`는 별도의 백그라운드 프로세스를 시작하여 다른 서비스 프로세스를 관리하기 때문에 `daemonize`가 필요하지 않습니다. 반면에 `daemonize => true`를 사용하면 `Swoole` 프로그램이 해당 관리 프로세스와 부모-자식 프로세스 관계를 잃을 수 있습니다.
### backlog

?> **`Listen` queue length 설정**

?> `backlog => 128`과 같이 설정하면 최대로 `accept`를 기다리는 연결 수를 결정합니다.

  * **`TCP`의 `backlog`에 대해**

    ?> `TCP`는 세 번의 핸드셰이크 과정을 거칩니다. 클라이언트 `syn => 서버` `syn+ack => 클라이언트` `ack`. 서버가 클라이언트의 `ack`를 받으면 연결을 `accept queue`라는 큐에 넣습니다 (주석1).  
    큐의 크기는 `backlog` 매개변수 및 `somaxconn` 구성의 최솟값에 의해 결정되며, 최종 `accept queue` 크기를 확인하려면 `ss -lt` 명령을 사용할 수 있습니다. `Swoole`의 메인 프로세스가 `accept`를 호출하면 (주석2) `accept queue`에서 연결을 가져옵니다. `accept queue`가 가득 찬 경우 연결이 성공할 수도 있습니다 (주석4),  
    실패할 수도 있으며, 실패한 경우 클라이언트의 동작은 연결이 재설정되거나 시간 초과됨을 의미합니다 (주석3).  
    서버는 실패 기록을 기록하며, `netstat -s|grep 'times the listen queue of a socket overflowed` 명령을 사용하여 로그를 확인할 수 있습니다. 위와 같은 현상이 발생하면 해당 값을 늘려야 합니다. 다행히도 `Swoole`의 SWOOLE_PROCESS 모드는 `PHP-FPM/Apache`와 같이 `backlog`에 의존하지 않으므로 위와 같은 현상을 거의 경험하지 않습니다.

    * 주석1: `linux2.2` 이후, 핸드셰이크 과정은 `syn queue`와 `accept queue` 두 개의 큐로 구분되며, `syn queue`의 길이는 `tcp_max_syn_backlog`에 의해 결정됩니다.
    * 주석2: 고버전 커널은 `accept4`를 호출하여 한 번의 `set no block` 시스템 호출을 절약합니다.
    * 주석3: 클라이언트는 `syn+ack` 패킷을 받으면 연결이 성공했다고 생각하지만, 실제로 서버는 반 열린 상태에 있을 수 있으며, 클라이언트에게 `rst` 패킷을 보내기도 합니다. 클라이언트의 동작은 `Connection reset by peer`입니다.
    * 주석4: 성공은 TCP의 재전송 메커니즘을 통해 이루어지며, 관련 구성은 `tcp_synack_retries`와 `tcp_abort_on_overflow`가 있습니다. TCP의 기본 메커니즘을 깊이 이해하려면 [Swoole 공식 비디오 강좌](https://course.swoole-cloud.com/course-video/3)를 참조하세요.
### open_tcp_keepalive

`TCP`에는 끊어진 연결을 감지하는 `Keep-Alive` 메커니즘이 있습니다. 응용 프로그램이 데드 링크 주기에 민감하지 않거나 하트비트 메커니즘을 구현하지 않은 경우, 운영 체제에서 제공하는 `keepalive` 메커니즘을 사용하여 끊어진 연결을 끊을 수 있습니다.
[서버 설정](/server/methods?id=set)에 `open_tcp_keepalive => true`를 추가하여 `TCP keepalive`를 활성화합니다.
또한, `keepalive`의 세부 사항을 조정하는 데 사용할 수 있는 `3`가지 옵션이 있습니다. [Swoole 공식 비디오 강의](https://course.swoole-cloud.com/course-video/10)를 참조하세요.

  * **옵션**

     * **tcp_keepidle**

        초 단위로, `n`초 동안 데이터 요청이 없는 연결에 대해 연결을 탐지하기 시작합니다.

     * **tcp_keepcount**

        탐지 시도 횟수를 초과하면이 연결을 `close`합니다.

     * **tcp_keepinterval**

        탐색 간격 시간, 단위 초.

  * **예시**

```php
$serv = new Swoole\Server("192.168.2.194", 6666, SWOOLE_PROCESS);
$serv->set(array(
    'worker_num' => 1,
    'open_tcp_keepalive' => true,
    'tcp_keepidle' => 4, //4초간 데이터 전송이 없으면 탐지 시작
    'tcp_keepinterval' => 1, //1초마다 탐지
    'tcp_keepcount' => 5, //탐지 시도 횟수, 5회 이상 응답이 없으면 연결 close
));

$serv->on('connect', function ($serv, $fd) {
    var_dump("Client:Connect $fd");
});

$serv->on('receive', function ($serv, $fd, $reactor_id, $data) {
    var_dump($data);
});

$serv->on('close', function ($serv, $fd) {
  var_dump("close fd $fd");
});

$serv->start();
```
### heartbeat_check_interval

?> **하트비트 체크 활성화**【기본값: `false`】

?> 이 옵션은 주기적으로 체크하는 간격을 나타내며, 단위는 초입니다. 예를 들어 `heartbeat_check_interval => 60`은 매 `60`초마다 모든 연결을 순회하며, 해당 연결이 `120`초 내에(만약 `heartbeat_idle_time`이 설정되지 않았다면 기본값은 `interval`의 두 배임) 서버로부터 어떠한 데이터도 보내지 않았을 경우 해당 연결을 강제로 닫히게 합니다. 이 옵션이 설정되어 있지 않으면 하트비트가 활성화되지 않으며, 이 설정은 기본적으로 비활성화되어 있습니다. [Swoole 공식 비디오 강의 참고](https://course.swoole-cloud.com/course-video/10)。

  * **팁**
    * `Server`는 클라이언트에게 하트비트를 주기적으로 보내지 않으며, 클라이언트로부터 하트비트를 기다립니다. 서버 측의 `heartbeat_check`는 단순히 연결이 마지막으로 데이터를 보낸 시간을 검사하며, 제한 시간을 초과하면 연결을 끊습니다.
    * 하트비트 체크로 인해 끊기는 연결은 여전히 [onClose](/server/events?id=onclose) 이벤트 콜백을 트리거합니다.

  * **주의**

    !> `heartbeat_check`은 `TCP` 연결만 지원합니다.
### heartbeat_idle_time

?> **최대 허용 대기 시간**

?> `heartbeat_check_interval`과 함께 사용해야 합니다.

```php
array(
    'heartbeat_idle_time'      => 600, // 연결이 600초 동안 서버로 데이터를 보내지 않으면 연결이 강제로 종료됩니다.
    'heartbeat_check_interval' => 60,  // 매 60초마다 확인합니다.
);
```

  * **팁**

    * `heartbeat_idle_time`을 활성화한 후 서버는 클라이언트로 데이터 패킷을 전송하지 않습니다.
    * `heartbeat_idle_time`만 설정하고 `heartbeat_check_interval`을 설정하지 않으면 하부에 하트비트 검사 스레드가 생성되지 않으며, `PHP` 코드에서 `heartbeat` 메소드를 호출하여 수동으로 시간 초과된 연결을 처리할 수 있습니다.
### open_eof_check

?> **`EOF` 체크 열기**【기본값: `false`】，[TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터패킷경계문제)를 참조하십시오.

?> 이 옵션은 클라이언트 연결에서 수신한 데이터를 확인하여 데이터 패킷 끝이 지정된 문자열인 경우에만 'Worker' 프로세스에 전달합니다. 그렇지 않으면 데이터 패킷을 계속 연결하여 버퍼를 초과하거나 시간 초과되기 전까지 기다립니다. 오류 발생 시 하위 수준에서는 악의적인 연결로 간주하여 데이터를 폐기하고 연결을 강제로 닫습니다.  
일반적으로 `Memcache/SMTP/POP`과 같은 프로토콜은 `\r\n`로 끝나므로 이 구성을 사용할 수 있습니다. 활성화하면 `Worker` 프로세스가 한 번에 하나 이상의 완전한 데이터 패킷을 항상 수신한다는 것을 보장할 수 있습니다.

```php
array(
    'open_eof_check' => true,   //EOF 체크 열기
    'package_eof'    => "\r\n", //EOF 설정
)
```

  * **주의**

    !> 이 구성은 `STREAM` 유형의 `Socket`에만 적용되며, [TCP、Unix Socket Stream](/server/methods?id=__construct)와 같은 것입니다.  
    `EOF` 체크가 데이터의 중간에서 `eof` 문자열을 찾지 않기 때문에 `Worker` 프로세스가 동시에 여러 데이터 패킷을 수신할 수 있으며, 응용 프로그램 코드에서 직접 `explode("\r\n", $data)`하여 데이터 패킷을 분할해야 합니다.
### open_eof_split

?> **`EOF` 자동 패킷 분할 활성화**

?> `open_eof_check`를 설정하면 하나의 패킷 내에 여러 데이터가 결합될 수 있습니다. `open_eof_split` 매개변수를 사용하여이 문제를 해결할 수 있습니다. 자세한 내용은 [TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터-패킷-경계-문제)를 참조하십시오.

?> 이 매개변수를 설정하면 CPU 리소스 소비가 많은 전체 데이터 패킷을 탐색하여 `EOF`를 찾아야 합니다. 각 데이터 패킷이 `2M`이고 매초 `10000`개의 요청이 들어오는 경우, CPU 문자 일치 명령이 `20G`개 생성될 수 있습니다.

```php
array(
    'open_eof_split' => true,   // EOF_SPLIT 검사 활성화
    'package_eof'    => "\r\n", // EOF 설정
)
```

  * **팁**

    * `open_eof_split` 매개변수를 활성화하면 하위 레벨에서 데이터 패킷 중간에서 `EOF`를 찾아 분할합니다. [onReceive](/server/events?id=onreceive)에서는 한 번에 하나씩 `EOF` 문자열로 끝나는 데이터 패킷만 받습니다.
    * `open_eof_split` 매개변수를 활성화하면 `open_eof_check` 매개변수가 설정되어 있는지 여부에 상관없이 `open_eof_split`이 동작합니다.

    * ** `open_eof_check` 와의 차이점**
    
        * `open_eof_check`는 수신 데이터의 끝이 `EOF`인지만 확인하므로 성능이 가장 우수하며 거의 리소스를 소비하지 않습니다.
        * `open_eof_check`는 여러 데이터 패킷이 결합되는 문제를 해결할 수 없습니다. 즉, `EOF`가 포함된 두 데이터를 동시에 보내면 하위 레벨에서 모두 한 번에 반환될 수 있습니다.
        * `open_eof_split`는 데이터를 왼쪽에서 오른쪽으로 바이트 단위로 비교하여 `EOF`를 찾아 패킷을 분할하므로 성능이 낮습니다. 그러나 매번 하나의 데이터 패킷만 반환됩니다.
### package_eof

?> **`EOF` 문자열을 설정합니다.** [TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터패킷경계문제)를 참조하세요.

?> `open_eof_check` 또는 `open_eof_split`와 함께 사용해야 합니다.

  * **주의**

    !> `package_eof`에는 최대 `8`바이트의 문자열만 허용됩니다.
### open_length_check

?> **패킷 길이 확인 기능**을 여십시오【기본값: `false`】，[TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터패킷경계문제)를 참조하십시오.

?> 길이 확인은 고정된 패킷 헤더 + 패킷 본문 형식의 프로토콜 해석을 제공합니다. 이 기능을 활성화하면 `Worker` 프로세스는[onReceive](/server/events?id=onreceive)에서 항상 완전한 데이터 패킷을 수신할 수 있습니다.  
길이 확인 프로토콜을 사용하면 한 번 길이를 계산하면 데이터 처리는 포인터 이동만 수행하여 매우 높은 성능을 제공하며 **사용 권장**됩니다.

  * **팁**

    * **길이 프로토콜은 프로토콜 세부 사항을 제어하기 위해 3가지 옵션을 제공합니다.**

      ?> 이 구성은 `STREAM` 유형의 `Socket`에만 유효하며, [TCP, Unix Socket Stream](/server/methods?id=__construct)과 같은 것입니다.

      * **package_length_type**

        ?> 패킷 헤더의 특정 필드를 패킷 길이 값으로 사용하며, 하위 수준에서 10가지 길이 유형을 지원합니다. 자세한 내용은 [package_length_type](/server/setting?id=package_length_type)을 참조하십시오.

      * **package_body_offset**

        ?> 몇 번째 바이트부터 길이를 계산할지 지정합니다. 일반적으로 2가지 경우가 있습니다:

        * `length` 값에는 전체 패킷(헤더 + 본문)이 포함되어 있으며, `package_body_offset`이 `0`입니다.
        * 패킷 헤더 길이가 `N`바이트이며, `length` 값에는 패킷 헤더가 아닌 본문만 포함되어 있습니다. `package_body_offset`을 `N`으로 설정합니다.

      * **package_length_offset**

        ?> `length` 길이 값이 패킷 헤더의 몇 번째 바이트에 있는지를 지정합니다.

        * 예시:

        ```c
        struct
        {
            uint32_t type;
            uint32_t uid;
            uint32_t length;
            uint32_t serid;
            char body[0];
        }
        ```
        
    ?> 위의 통신 프로토콜 설계에서 패킷 헤더 길이는 `4`개의 정수 및 `16`바이트입니다. `length` 길이 값은 3번째 정수 위치에 있습니다. 따라서 `package_length_offset`은 `8`로 설정되며, `0-3`바이트는 `type`, `4-7`바이트는 `uid`, `8-11`바이트는 `length`, `12-15`바이트는 `serid`입니다.

    ```php
    $server->set(array(
      'open_length_check'     => true,
      'package_max_length'    => 81920,
      'package_length_type'   => 'N',
      'package_length_offset' => 8,
      'package_body_offset'   => 16,
    ));
    ```
### package_length_type

?> **길이 값의 유형**, 한 문자 매개변수를 사용하며 `PHP`의 [pack](http://php.net/manual/zh/function.pack.php) 함수와 동일합니다.

현재 `Swoole`은 `10`가지 유형을 지원합니다:

문자 매개변수 | 기능
---|---
c | 부호 있는, 1바이트
C | 부호 없는, 1바이트
s | 부호 있는, 호스트 바이트 순서, 2바이트
S | 부호 없는, 호스트 바이트 순서, 2바이트
n | 부호 없는, 네트워크 바이트 순서, 2바이트
N | 부호 없는, 네트워크 바이트 순서, 4바이트
l | 부호 있는, 호스트 바이트 순서, 4바이트 (소문자 L)
L | 부호 없는, 호스트 바이트 순서, 4바이트 (대문자 L)
v | 부호 없는, 리틀 엔디안 바이트 순서, 2바이트
V | 부호 없는, 리틀 엔디안 바이트 순서, 4바이트
### package_length_func

?> **길이 구문 함수 설정**

?> `C++` 또는 `PHP`의 두 가지 유형의 함수를 지원합니다. 길이 함수는 정수를 반환해야 합니다.

반환 값 | 역할
---|---
0을 반환 | 길이 데이터가 충분하지 않아 추가 데이터를 수신해야 함
-1을 반환 | 데이터 오류로 인해 하부에서 연결이 자동으로 닫힘
패키지 길이 값(패키지 헤더와 본문의 총 길이 포함)을 반환 | 하부에서 패키지를 자동으로 조립한 후 콜백 함수에 반환

  * **팁**

    * **사용 방법**

    ?> 구현 원리는 먼저 작은 부분 데이터를 읽어서 이 데이터에 길이 값을 포함시킵니다. 그런 다음이 길이를 하위에 반환하고 나머지 데이터 수집 및 패키지로의 조립을 `dispatch`로 완료하게 합니다.

    * **PHP 길이 구문 함수**

    ?> `ZendVM`이 멀티 스레드 환경에서 실행되지 않기 때문에 하부에서 `PHP` 길이 함수에 대한 경합 상태를 피하기 위해 자동으로 `Mutex` 뮤텍스 잠금을 사용합니다. `1.9.3` 이상에서 사용할 수 있습니다.

    !> 길이 구문 함수에서 블로킹 `IO` 작업을 수행하지 마십시오. 이로 인해 모든 [Reactor](/learn?id=reactor-line) 스레드가 차단될 수 있습니다.

    ```php
    $server = new Swoole\Server("127.0.0.1", 9501);
    
    $server->set(array(
        'open_length_check'   => true,
        'dispatch_mode'       => 1,
        'package_length_func' => function ($data) {
          if (strlen($data) < 8) {
              return 0;
          }
          $length = intval(trim(substr($data, 0, 8)));
          if ($length <= 0) {
              return -1;
          }
          return $length + 8;
        },
        'package_max_length'  => 2000000,  //프로토콜 최대 길이
    ));
    
    $server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        var_dump($data);
        echo "#{$server->worker_id}>> received length=" . strlen($data) . "\n";
    });
    
    $server->start();
    ```

    * **C++ 길이 구문 함수**

    ?> 다른 PHP 확장에서는 `swoole_add_function`을 사용하여 길이 함수를 `Swoole` 엔진에 등록합니다.
    
    !> C++ 길이 함수는 호출될 때 하부에서 잠금이 적용되지 않습니다. 호출자가 스레드 안전성을 보장해야 합니다.
    
    ```c++
    #include <string>
    #include <iostream>
    #include "swoole.h"
    
    using namespace std;
    
    int test_get_length(swProtocol *protocol, swConnection *conn, char *data, uint32_t length);
    
    void register_length_function(void)
    {
        swoole_add_function((char *) "test_get_length", (void *) test_get_length);
        return SW_OK;
    }
    
    int test_get_length(swProtocol *protocol, swConnection *conn, char *data, uint32_t length)
    {
        printf("cpp, size=%d\n", length);
        return 100;
    }
    ```
### package_max_length

?> **최대 데이터 패킷 크기를 바이트 단위로 설정합니다.**\[기본값: `2M`, 즉 `2 * 1024 * 1024`, 최소값은 `64K`\]

?> [open_length_check](/server/setting?id=open_length_check)/[open_eof_check](/server/setting?id=open_eof_check)/[open_eof_split](/server/setting?id=open_eof_split)/[open_http_protocol](/server/setting?id=open_http_protocol)/[open_http2_protocol](/http_server?id=open_http2_protocol)/[open_websocket_protocol](/server/setting?id=open_websocket_protocol)/[open_mqtt_protocol](/server/setting?id=open_mqtt_protocol)과 같은 프로토콜 parsing 후, `Swoole`의 하부에서 데이터 패킷을 조립하게 되며, 패킷이 완전히 수신되지 않은 경우 모든 데이터는 메모리에 저장됩니다.  
그래서 `package_max_length`를 설정해야 합니다. 데이터 패킷이 허용하는 최대 메모리 크기입니다. 만약 1만개의 `TCP` 연결이 동시에 데이터를 보내고, 각 데이터 패킷이` 2M`이면, 최악의 경우 메모리 공간은 `20G`를 차지할 수 있습니다.

  * **팁**

    * `open_length_check` : 패킷 길이가 `package_max_length`를 초과하는 경우 데이터가 바로 삭제되고 연결이 닫힙니다. 어떤 메모리도 사용되지 않습니다.
    * `open_eof_check` : 데이터 패킷 길이를 알 수 없으므로 데이터가 계속해서 메모리에 저장됩니다. 메모리 사용량이 `package_max_length`를 초과하면 해당 데이터가 삭제되고 연결이 닫힙니다.
    * `open_http_protocol` : `GET` 요청은 최대 `8K`를 허용하며 설정을 수정할 수 없습니다. `POST` 요청은 `Content-Length`를 확인하며, 만약 `Content-Length`가 `package_max_length`를 초과하면 해당 데이터가 삭제되고 `http 400` 오류가 발생하며 연결이 닫힙니다.

  * **주의**

    !> 이 매개변수는 너무 크게 설정되면 많은 메모리를 사용하므로 권장되지 않습니다.
### open_http_protocol

?> **`HTTP` 프로토콜을 처리하도록 설정합니다.**【기본값: `false`】

?> `HTTP` 프로토콜을 처리하도록 설정하면 [Swoole\Http\Server](/http_server)가 자동으로 이 옵션을 활성화합니다. `false`로 설정하면 `HTTP` 프로토콜 처리가 비활성화됩니다.
### open_mqtt_protocol

?> **`MQTT` 프로토콜 처리를 활성화합니다.**【기본값: `false`】

?> 활성화하면 `MQTT` 패킷 헤더를 해석하며 `worker` 프로세스가 [onReceive](/server/events?id=onreceive)를 호출할 때마다 완전한 `MQTT` 데이터 패킷을 반환합니다.

```php
$server->set(array(
  'open_mqtt_protocol' => true
));
```
### open_redis_protocol

?> **Redis 프로토콜 처리를 활성화합니다.**【기본값: `false`】

?> 활성화하면 Redis 프로토콜을 해석하며, worker 프로세스는 [onReceive](/server/events?id=onreceive)마다 완전한 Redis 데이터 패킷을 반환합니다. [Redis\Server](/redis_server)를 직접 사용하는 것이 좋습니다.

```php
$server->set(array(
  'open_redis_protocol' => true
));
```
### open_websocket_protocol

?> **`WebSocket` 프로토콜 처리를 활성화합니다.**【기본값: `false`】

?> `WebSocket` 프로토콜 처리를 활성화하면 [Swoole\WebSocket\Server](websocket_server)가 자동으로 이 옵션을 활성화합니다. `false`로 설정하면 `websocket` 프로토콜 처리가 비활성화됩니다.  
`open_websocket_protocol` 옵션을 `true`로 설정하면 `open_http_protocol` 프로토콜도 자동으로 `true`로 설정됩니다.
### open_websocket_close_frame

?> **웹소켓 프로토콜에서 닫는 프레임을 활성화합니다.**【기본값: `false`】

?> (`opcode`가 `0x08`인 프레임)은 `onMessage` 콜백에서 받습니다.

?> 활성화하면 `WebSocketServer`에서 클라이언트 또는 서버로부터 전송된 닫는 프레임을 `onMessage` 콜백에서 받을 수 있으며, 개발자는 필요에 따라 처리할 수 있습니다.

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->set(array("open_websocket_close_frame" => true));

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x08) {
        echo "Close frame received: Code {$frame->code} Reason {$frame->reason}\n";
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {});

$server->start();
```
### open_tcp_nodelay

?> **`open_tcp_nodelay`를 활성화합니다.**【기본값: `false`】

?> 활성화하면 데이터를 보낼 때 TCP 연결이 Nagle 결합 알고리즘을 비활성화하고 즉시 상대편 TCP 연결로 전송합니다. 일부 시나리오에서는 명령줄 터미널과 같이 명령을 즉시 서버로 전송해야 하는 경우 응답 속도를 높일 수 있습니다. Nagle 알고리즘에 대해 직접 Google에서 확인하십시오.
### open_cpu_affinity 

?> **CPU 친화성 설정을 활성화합니다.** 【기본값 `false`】

?> 다중 코어 하드웨어 플랫폼에서이 기능을 활성화하면 `Swoole`의 `리액터 스레드`/`작업자 프로세스`가 특정 코어에 바인딩됩니다. 프로세스/스레드가 다중 코어 간에 상호 전환되는 것을 방지하여 `CPU` 캐시의 적중률을 향상시킵니다.

  * **팁**

    * **taskset 명령어를 사용하여 프로세스의 CPU 친화도 설정을 확인하십시오：**

    ```bash
    taskset -p 프로세스ID
    pid 24666's current affinity mask: f
    pid 24901's current affinity mask: 8
    ```

    > mask는 비트 단위로 CPU 코어에 해당하는 비트를 나타내는 마스크 숫자입니다. 특정 비트가 `0`이면 해당 코어에 바인딩되어 프로세스가 해당 CPU로 스케줄됩니다. 다른 경우는 해당 CPU에 프로세스가 스케줄되지 않음을 나타냅니다. 예에서 `pid`가 `24666`인 프로세스의 `mask = f`는 CPU에 바인딩되어 있지 않음을 나타냅니다. 운영 체제는이 프로세스를 임의의 CPU 코어에 스케줄합니다. `pid`가 `24901`의 프로세스인 경우 `mask = 8`이며, `8`을 이진으로 변환하면` 1000`으로, 이로써 이 프로세스가 4 번째 CPU 코어에 바인딩되어 있다는 것을 나타냅니다.
### cpu_affinity_ignore

?> **IO-intensive programs have all network interrupts processed by CPU0. If network IO is heavy and CPU0 is overloaded, network interrupts may not be handled in a timely manner, resulting in a decrease in network packet processing capacity.**

?> If this option is not set, swoole will use all CPU cores, and the underlying system will set CPU binding based on reactor_id or worker_id modulo the number of CPUs.
If there are multi-queue features between the kernel and the network card, network interrupts will be distributed across multiple cores, which helps alleviate the pressure on network interrupts.

```php
array('cpu_affinity_ignore' => array(0, 1)) // Accepts an array as a parameter, where array(0, 1) means that CPU0 and CPU1 are not used and are left empty to handle network interrupts separately.
```

  * **Hint**

    * **Check network interrupts**

```shell
[~]$ cat /proc/interrupts 
           CPU0       CPU1       CPU2       CPU3       
  0: 1383283707          0          0          0    IO-APIC-edge  timer
  1:          3          0          0          0    IO-APIC-edge  i8042
  3:         11          0          0          0    IO-APIC-edge  serial
  8:          1          0          0          0    IO-APIC-edge  rtc
  9:          0          0          0          0   IO-APIC-level  acpi
 12:          4          0          0          0    IO-APIC-edge  i8042
 14:         25          0          0          0    IO-APIC-edge  ide0
 82:         85          0          0          0   IO-APIC-level  uhci_hcd:usb5
 90:         96          0          0          0   IO-APIC-level  uhci_hcd:usb6
114:    1067499          0          0          0       PCI-MSI-X  cciss0
130:   96508322          0          0          0         PCI-MSI  eth0
138:     384295          0          0          0         PCI-MSI  eth1
169:          0          0          0          0   IO-APIC-level  ehci_hcd:usb1, uhci_hcd:usb2
177:          0          0          0          0   IO-APIC-level  uhci_hcd:usb3
185:          0          0          0          0   IO-APIC-level  uhci_hcd:usb4
NMI:      11370       6399       6845       6300 
LOC: 1383174675 1383278112 1383174810 1383277705 
ERR:          0
MIS:          0
```

`eth0/eth1` represents the number of network interrupts. If `CPU0 - CPU3` evenly distributes the interrupts, it indicates that the network card has multi-queue features. If all interrupts are concentrated on one core, it means all network interrupts are handled by that CPU. Once this CPU exceeds 100%, the system won't be able to process network requests. In such cases, the `cpu_affinity_ignore` option should be used to designate this CPU exclusively for handling network interrupts, leaving it empty.

Based on the scenario above, `cpu_affinity_ignore => array(0)` should be set.

?> You can use the `top` command `->` type `1` to view the usage of each core.

  * **Note**

    !> This option must be set along with `open_cpu_affinity` to be effective.
### tcp_defer_accept

?> **`tcp_defer_accept` 특성 활성화**【기본값: `false`】

?> 숫자로 설정하여, `TCP` 연결이 데이터를 보낼 때만 `accept`를 트리거할 수 있습니다.

```php
$server->set(array(
  'tcp_defer_accept' => 5
));
```

  * **팁**

    * **`tcp_defer_accept` 특성을 활성화한 후, `accept`와[onConnect](/server/events?id=onconnect)에 대한 시간이 변경됩니다. `5`로 설정한 경우:**

      * 서버에 클라이언트가 연결되면 즉시 `accept`가 트리거되지 않습니다.
      * 클라이언트가 `5`초 내에 데이터를 보내면 동시에 `accept/onConnect/onReceive`가 순서대로 트리거됩니다.
      * 클라이언트가 `5`초 내에 데이터를 보내지 않으면 `accept/onConnect`가 트리거됩니다.
### ssl_cert_file / ssl_key_file :id=ssl_cert_file

?> **SSL 터널 암호화 설정.**

?> 파일명 문자열 값으로 지정된 cert 인증서 및 키파일의 경로를 나타냅니다.

  * **팁**

    * **`PEM`을 `DER` 형식으로 변환**

    ```shell
    openssl x509 -in cert.crt -outform der -out cert.der
    ```

    * **`DER`를 `PEM` 형식으로 변환**

    ```shell
    openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
    ```

  * **주의**

    !> - `HTTPS` 애플리케이션은 브라우저가 인증서를 신뢰해야만 웹 페이지를 볼 수 있습니다;
    - `wss` 애플리케이션에서 `WebSocket` 연결을 시작하는 페이지는 `HTTPS`를 사용해야 합니다;
    - 브라우저가 `SSL` 인증서를 신뢰하지 않으면 `wss`를 사용할 수 없습니다;
    - 파일은 `PEM` 형식이어야 하며 `DER` 형식은 지원되지 않습니다. `openssl` 도구를 사용하여 변환할 수 있습니다.

    !> `SSL`을 사용하려면 `Swoole`을 컴파일할 때 [--enable-openssl](/environment?id=编译选项) 옵션을 추가해야 합니다

    ```php
    $server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
    $server->set(array(
        'ssl_cert_file' => __DIR__.'/config/ssl.crt',
        'ssl_key_file' => __DIR__.'/config/ssl.key',
    ));
    ```
### ssl_method

!> This parameter has been removed in [v4.5.4](/version/bc?id=_454). Please use `ssl_protocols` instead.

?> **Set the encryption algorithm for OpenSSL tunnel.** 【Default value: `SWOOLE_SSLv23_METHOD`】, Refer to [SSL Encryption Method](/consts?id=ssl-encryption-method) for supported types.

?> The algorithms used by both `Server` and `Client` must be consistent, otherwise `SSL/TLS` handshake will fail, and the connection will be terminated.

```php
$server->set(array(
    'ssl_method' => SWOOLE_SSLv3_CLIENT_METHOD,
));
```
### ssl_protocols

?> **OpenSSL 터널 암호화 프로토콜을 설정합니다.** [기본값: `0`, 모든 프로토콜 지원], 지원되는 유형은 [SSL 프로토콜](/consts?id=ssl-프로토콜)을 참조하세요.

!> Swoole 버전 >= `v4.5.4`에서 사용 가능

```php
$server->set(array(
    'ssl_protocols' => 0,
));
```
### ssl_sni_certs

?> **SNI (Server Name Identification) 인증서 설정**

!> Swoole 버전 >= `v4.6.0`에서 사용 가능

```php
$server->set([
    'ssl_cert_file' => __DIR__ . '/server.crt',
    'ssl_key_file' => __DIR__ . '/server.key',
    'ssl_protocols' => SWOOLE_SSL_TLSv1_2 | SWOOLE_SSL_TLSv1_3 | SWOOLE_SSL_TLSv1_1 | SWOOLE_SSL_SSLv2,
    'ssl_sni_certs' => [
        'cs.php.net' => [
            'ssl_cert_file' => __DIR__ . '/sni_server_cs_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_cs_key.pem',
        ],
        'uk.php.net' => [
            'ssl_cert_file' =>  __DIR__ . '/sni_server_uk_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_uk_key.pem',
        ],
        'us.php.net' => [
            'ssl_cert_file' => __DIR__ . '/sni_server_us_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_us_key.pem',
        ],
    ]
]);
```
### ssl_ciphers

?> **SSL 암호를 설정합니다.** [기본값: `EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH`]

```php
$server->set(array(
    'ssl_ciphers' => 'ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP',
));
```

  * **팁**

    * `ssl_ciphers`를 빈 문자열로 설정하면 `openssl`이 자체적으로 암호화 알고리즘을 선택합니다.
### ssl_verify_peer

?> **상대 SSL 인증을 검증하는 서비스를 설정합니다.** 【기본값: `false`】

?> 기본적으로 비활성화되어 있으며, 클라이언트 인증을 검증하지 않습니다. 활성화되면, `ssl_client_cert_file` 옵션도 설정해야 합니다.
### ssl_allow_self_signed

?> **자체 서명된 인증서를 허용합니다.**【기본값: `false`】
### ssl_client_cert_file

?> **클라이언트 인증서를 확인하는 데 사용되는 루트 인증서입니다.**

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set(array(
    'ssl_cert_file'         => __DIR__ . '/config/ssl.crt',
    'ssl_key_file'          => __DIR__ . '/config/ssl.key',
    'ssl_verify_peer'       => true,
    'ssl_allow_self_signed' => true,
    'ssl_client_cert_file'  => __DIR__ . '/config/ca.crt',
));
```

!> `TCP` 서비스에서 검증에 실패할 경우, 하위 연결을 자동으로 닫습니다.
### ssl_compress

?> **`SSL/TLS` 압축을 사용 여부를 설정합니다.** [Co\Client](/coroutine_client/client)에서 사용할 때 `ssl_disable_compression`이라는 별칭이 있습니다.
### ssl_verify_depth

?> **인증서 체인의 깊이가이 옵션 값보다 깊은 경우 검증이 중단됩니다.**
### ssl_prefer_server_ciphers

?> **서버 측 암호를 우선 사용하여 BEAST 공격을 방지합니다.**
### ssl_dhparam

?> **`Diffie-Hellman` 파라미터에 대한 `DHE` 암호기를 지정합니다.**
### ssl_ecdh_curve

?> **ECDH 키 교환에 사용되는 `curve`를 지정합니다.**

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set([
    'ssl_compress'                => true,
    'ssl_verify_depth'            => 10,
    'ssl_prefer_server_ciphers'   => true,
    'ssl_dhparam'                 => '',
    'ssl_ecdh_curve'              => '',
]);
```
### 사용자

?> **`Worker/TaskWorker` 자식 프로세스의 소유자 설정.**【기본값: 스크립트를 실행하는 사용자】

?> 서버가 `1024` 미만의 포트를 수신해야 하는 경우 `root` 권한이 필요합니다. 그러나 프로그램은 `root` 사용자로 실행되면 코드에 취약점이 있는 경우 공격자가 원격 명령을 `root`로 실행할 수 있어 큰 위험이 됩니다. `user` 옵션을 구성하면 메인 프로세스를 `root` 권한으로 실행하고 하위 프로세스를 일반 사용자 권한으로 실행할 수 있습니다.

```php
$server->set(array(
  'user' => 'Apache'
));
```

  * **주의**

    !> -`root` 사용자로 시작할 때에만 유효함  
    -`user/group` 구성을 사용하여 작업 프로세스를 일반 사용자로 설정하면 작업 프로세스에서 `shutdown`/[reload](/server/methods?id=reload) 방법을 통해 서비스를 종료하거나 다시 시작할 수 없습니다. `shell` 터미널에서 `root` 계정으로 `kill` 명령을 실행해야 합니다.
### 그룹

?> **`Worker/TaskWorker` 하위 프로세스의 프로세스 사용자 그룹 설정.**【기본 값: 스크립트를 실행하는 사용자 그룹】

?> `user` 설정과 동일하며, 이 설정은 프로세스가 속한 그룹을 변경하여 서버 프로그램의 보안을 강화합니다.

```php
$server->set(array(
  'group' => 'www-data'
));
```

  * **주의**

    !> `root` 사용자로 시작했을 때에만 유효합니다
### chroot

?> **`Worker` 프로세스의 파일 시스템 루트를 재지정합니다.**

?> 이 설정은 프로세스가 파일 시스템을 읽고 쓰는 것을 실제 OS 파일 시스템과 격리시켜 안전성을 향상시킵니다.

```php
$server->set(array(
  'chroot' => '/data/server/'
));
```
### pid_file

?> **pid 파일 경로를 설정합니다.**

?> `Server`를 시작할 때 `master` 프로세스의 `PID`를 자동으로 파일에 작성하고, `Server`를 종료할 때 `PID` 파일을 자동으로 삭제합니다.

```php
$server->set(array(
    'pid_file' => __DIR__.'/server.pid',
));
```

  * **참고**

    !> 비정상적인 경우에 `Server`가 종료되면 `PID` 파일이 삭제되지 않으므로, 프로세스가 실제로 존재하는지 확인하려면 [Swoole\Process::kill($pid, 0)](/process/process?id=kill)를 사용해야 합니다.
### buffer_input_size / input_buffer_size :id=buffer_input_size

?> **입력 버퍼 크기를 설정합니다.**【기본값 : `2M`】

```php
$server->set([
    'buffer_input_size' => 2 * 1024 * 1024,
]);
```
### buffer_output_size / output_buffer_size :id=buffer_output_size

?> **출력 버퍼 크기를 설정합니다.**【기본값: `2M`】

```php
$server->set([
    'buffer_output_size' => 32 * 1024 * 1024, // 숫자 여야 함
]);
```  

  * **팁**

    !> Swoole 버전 >= `v4.6.7` 일 때, 기본값은 부호 없는 INT 최대값 `UINT_MAX` 입니다.

    * 단위는 바이트이며, 기본값은 `2M` 입니다. `32 * 1024 * 1024`로 설정하면, `Server->send` 한 번에 최대 `32M` 바이트 데이터를 전송할 수 있습니다.
    * `Server->send`, `Http\Server->end/write`, `WebSocket\Server->push`와 같은 데이터 전송 명령을 호출할 때, `한 번에` 보낼 수 있는 데이터의 최대 크기가 `buffer_output_size` 설정을 초과하지 않아야 합니다.

    !> 이 매개변수는 [SWOOLE_PROCESS](/learn?id=swoole_process) 모드에만 적용되며, 이는 PROCESS 모드에서 Worker 프로세스의 데이터가 클라이언트에 전송되기 전에 주 프로세스로 전송되어야하기 때문에, 각 Worker 프로세스가 주 프로세스에 대해 하나의 버퍼를 확보하게 됩니다. [참고](/learn?id=reactor线程)
### socket_buffer_size

?> **클라이언트 연결의 버퍼 크기를 설정합니다.**【기본값: `2M`】

?> `buffer_output_size`와는 다르게, `buffer_output_size`는 워커 프로세스의 `한 번`의 전송 크기 제한이며, `socket_buffer_size`는 `Worker`와 `Master` 프로세스 간 통신 버퍼의 총 크기를 설정하는 데 사용됩니다. [SWOOLE_PROCESS](/learn?id=swoole_process) 모드를 참조하십시오.

```php
$server->set([
    'socket_buffer_size' => 128 * 1024 *1024, // 바이트 단위의 숫자여야 합니다. 예: 128 * 1024 *1024는 각 TCP 클라이언트 연결당 최대 128M의 보류 중인 데이터가 허용됨을 의미합니다.
]);
```

- **데이터 전송 버퍼**

    - Master 프로세스가 대량의 데이터를 클라이언트에 보낼 때 즉시 전송되지 않을 수 있습니다. 이 때 보낸 데이터는 서버의 내부 메모리 버퍼에 저장됩니다. 이 매개변수는 내부 메모리 버퍼의 크기를 조절합니다.
    
    - 데이터가 너무 많이 전송되면 서버가 다음과 같은 오류 메시지를 표시할 수 있습니다:
    
    ```bash
    swFactoryProcess_finish: send failed, session#1 output buffer has been overflowed.
    ```

    ?> 전송 버퍼가 가득 차서 `send`에 실패하면 현재 클라이언트에만 영향을 미치며, 다른 클라이언트에는 영향이 없습니다. 서버에 많은 `TCP` 연결이 있으면 최악의 경우 `serv->max_connection * socket_buffer_size` 바이트의 메모리를 사용할 수 있습니다.

    - 특히 네트워크 통신이 느린 서버 프로그램의 경우 지속적으로 데이터를 전송하면 버퍼가 빠르게 가득 찰 수 있습니다. 전송된 데이터는 모두 서버 메모리에 쌓입니다. 따라서 이러한 응용 프로그램은 네트워크 전송 능력을 고려하여 설계해야 하며, 먼저 메시지를 디스크에 저장하고 클라이언트가 수신을 완료했다고 서버에 통보한 후 새 데이터를 전송해야 합니다.

    - 예를 들어, 비디오 스트리밍 서비스에서 `A` 사용자의 대역폭이 `100M`이면 `1`초에 `10M`의 데이터를 보내는 것은 완전히 가능합니다. `B` 사용자의 대역폭이 `1M`인 경우, `1`초에 `10M`의 데이터를 보내면 `B` 사용자가 완전히 수신하는 데 `100`초가 걸릴 수 있습니다. 이러한 경우 데이터가 서버 메모리에 모두 쌓입니다.

    - 데이터의 유형에 따라 다른 처리를 할 수 있습니다. 비디오 스트리밍과 같이 폐기할 수 있는 콘텐츠인 경우 네트워크 연결이 느린 경우 일부 데이터 프레임을 폐기하는 것은 완전히 수용될 수 있습니다. 메시지와 같이 폐기할 수 없는 내용인 경우, 예를 들어 `100`개의 메시지를 하나의 그룹으로 저장해 서버 디스크에 먼저 저장할 수 있습니다. 사용자가 이 그룹의 메시지를 완료하면 디스크에서 다음 그룹 메시지를 가져와 클라이언트에게 보낼 수 있습니다.
### enable_unsafe_event

?> **`onConnect/onClose` 이벤트를 활성화합니다.**【기본값: `false`】

?> `Swoole`은 설정 [dispatch_mode](/server/setting?id=dispatch_mode)=1 또는 `3` 후, 시스템이 `onConnect/onReceive/onClose`의 순서를 보장할 수 없기 때문에 기본적으로 `onConnect/onClose` 이벤트가 비활성화됩니다;  
응용 프로그램이 `onConnect/onClose` 이벤트가 필요하고 순서 문제가 발생할 수 있는 보안 위험을 수용할 수 있다면, `enable_unsafe_event`를 `true`로 설정하여 `onConnect/onClose` 이벤트를 활성화할 수 있습니다.
### discard_timeout_request

?> **닫힌 연결의 데이터 요청을 폐기합니다.**【기본값: `true`】

?> `Swoole`은 [dispatch_mode](/server/setting?id=dispatch_mode)이 `1` 또는 `3`로 설정된 경우 `onConnect/onReceive/onClose`의 순서를 보장할 수 없으므로 연결이 닫힌 후에도 일부 요청 데이터가 `Worker` 프로세스에 도달할 수 있습니다.

  * **팁**

    * `discard_timeout_request` 구성은 기본적으로`true`로 설정되어 있으며, 만약 `worker` 프로세스가 이미 닫힌 연결의 데이터 요청을 받았다면 이를 자동으로 폐기합니다.
    * `discard_timeout_request`를 `false`로 설정하면 연결이 닫혀 있든 열려 있든 `Worker` 프로세스가 데이터 요청을 처리합니다.
### enable_reuse_port

?> **포트 재사용을 설정합니다.** [기본값: `false`]

?> 포트 재사용을 활성화하면 동일한 포트를 사용하여 서버 프로그램을 반복해서 시작할 수 있습니다.

  * **팁**

    * `enable_reuse_port = true` 포트 재사용을 엽니다.
    * `enable_reuse_port = false` 포트 재사용을 닫습니다.

!> `Linux-3.9.0` 버전 이상의 커널에서만 사용 가능 `Swoole4.5` 이상의 버전에서 사용 가능
### enable_delay_receive

?> **`accept` 클라이언트 연결 후에 [EventLoop](/learn?id=이벤트루프가 무엇인가요)에 자동으로 추가되지 않도록 설정됩니다.**【기본값: `false`】

?> 이 옵션을 `true`로 설정하면 `accept` 클라이언트 연결 후 [EventLoop](/learn?id=이벤트루프가 무엇인가요)에 자동으로 추가되지 않고, [onConnect](/server/events?id=onconnect) 콜백만 트리거됩니다. `worker` 프로세스는 [$server->confirm($fd)](/server/methods?id=confirm)를 호출하여 연결을 확인해야만 `fd`가 [EventLoop](/learn?id=이벤트루프가 무엇인가요)에 추가되어 데이터 송수신이 시작됩니다. 또한 `$server->close($fd)`를 호출하여 이 연결을 닫을 수도 있습니다.

```php
// enable_delay_receive 옵션 활성화
$server->set(array(
    'enable_delay_receive' => true,
));

$server->on("Connect", function ($server, $fd, $reactorId) {
    $server->after(2000, function() use ($server, $fd) {
        // 연결을 확인하고 데이터 수신 시작
        $server->confirm($fd);
    });
});
```
### reload_async

?> **비동기 재시작 스위치를 설정합니다.**【기본값: `true`】

?> 비동기 재시작 스위치를 설정합니다. `true`로 설정하면 비동기 안전 재시작 기능이 활성화되어 `Worker` 프로세스가 비동기 이벤트가 완료될 때까지 기다린 후 종료됩니다. 자세한 내용은 [서비스를 올바르게 다시 시작하는 방법](/question/use?id=swoole如何正确的重启服务)을 참조하십시오.

?> `reload_async`를 활성화하는 주요 목적은 서비스가 다시로드될 때 코루틴 또는 비동기 작업이 정상적으로 종료되도록 보장하는 것입니다.

```php
$server->set([
  'reload_async' => true
]);
```

  * **코루틴 모드**

    * `4.x` 버전에서 [enable_coroutine](/server/setting?id=enable_coroutine)을 활성화할 때, 내부에서 추가로 코루틴 수를 확인하고 아무 코루틴이 없을 때에만 프로세스가 종료됩니다. 이 모드를 활성화하면 `reload_async => false`로 설정되어 있어도 강제로 `reload_async`가 열리게 됩니다.
### max_wait_time

?> **`Worker` 프로세스가 서비스 중지 알림을 받은 후 최대 대기 시간 설정**【기본값: `3`】

?> 종종 `worker`가 블로킹되어 재시작이 불가능한 상황이 발생할 수 있으며, 이로 인해 일부 프로덕션 시나리오를 충족시키지 못할 수 있습니다. 예를 들어 코드 냉각 업데이트를 수행해야 할 때 `reload` 프로세스가 필요합니다. 그래서 Swoole은 프로세스 재시작 시간 초과 옵션을 추가했습니다. 자세한 정보는 [올바른 방법으로 서비스 재시작하는 방법](/question/use?id=swoole如何正确的重启服务)을 참조하세요.

  * **팁**

    * **관리 프로세스가 재시작 또는 종료 신호를 받거나 `max_request`에 도달한 경우 관리 프로세스는 해당 `worker` 프로세스를 재시작합니다. 다음 단계를 거칩니다：**

      * 하위 레벨에서 (`max_wait_time`)초의 타이머가 추가됩니다. 타이머가 트리거되면 프로세스의 존재 여부를 확인하고, 존재하는 경우 강제로 종료하고 새로운 프로세스를 시작합니다.
      * `onWorkerStop` 콜백에서 청소 작업을 수행해야 하며, 이는 `max_wait_time`초 내에 완료되어야 합니다.
      * 대상 프로세스에 차례로 `SIGTERM` 신호를 보내 프로세스를 종료합니다.

  * **주의**

    !> `v4.4.x` 이전에는 기본값이 `30`초였습니다.
### tcp_fastopen

?> **TCP 빠른 핸드셰이크 기능을 활성화합니다.**【기본값: `false`】

?> 이 기능은 `TCP` 단기 연결의 응답 시간을 개선할 수 있으며, 클라이언트가 핸드셰이크의 3단계를 완료한 후 `SYN` 패킷을 보낼 때 데이터를 전송합니다.

```php
$server->set([
  'tcp_fastopen' => true
]);
```

  * **팁**

    * 이 매개변수는 수신 대기 포트에 설정할 수 있으며, 깊이 이해하고 싶은 학생들은 [google 논문](http://conferences.sigcomm.org/co-next/2011/papers/1569470463.pdf)을 참조하십시오.
### request_slowlog_file

?> **요청 느린 로그를 활성화합니다.** `v4.4.8` 버전부터는 [제거되었습니다](https://github.com/swoole/swoole-src/commit/b1a400f6cb2fba25efd2bd5142f403d0ae303366)

!> 이 느린 로그 방식은 동기 블로킹 프로세스에서만 작동하며, 코루틴 환경에서는 사용할 수 없습니다. Swoole4는 기본적으로 코루틴이 활성화되어 있으므로 `enable_coroutine`을 비활성화하지 않는 한 사용하지 마십시오. 대신 [Swoole Tracker](https://business.swoole.com/tracker/index)의 블로킹 감지 도구를 사용하세요.

?> 활성화하면 `Manager` 프로세스가 타이머 신호를 설정하여 모든 `Task`와 `Worker` 프로세스를 주기적으로 감지하며, 프로세스가 블로킹되어 요청 시간 제한을 초과하는 경우 자동으로 프로세스의 `PHP` 함수 호출 스택을 출력합니다.

?> 이것은 `ptrace` 시스템 호출을 기반으로 한 것이며, 일부 시스템은 `ptrace`가 비활성화될 수 있어 느린 요청을 추적할 수 없을 수 있습니다. `kernel.yama.ptrace_scope` 커널 매개변수가 `0`인지 확인해주세요.

```php
$server->set([
  'request_slowlog_file' => '/tmp/trace.log',
]);
```

  * **시간 초과 시간**

```php
$server->set([
    'request_slowlog_timeout' => 2, // 요청 시간 초과 시간을 2초로 설정
    'request_slowlog_file' => '/tmp/trace.log',
]);
```

!> 꼭 쓰기 권한이 있는 파일이어야 하며, 그렇지 않으면 파일 생성에 실패하여 하위 레벨에서 치명적인 오류가 발생할 수 있습니다.
### enable_coroutine

?> **비동기 스타일 서버의 코루틴 지원 활성화**

?> `enable_coroutine`이 꺼져 있으면 [이벤트 콜백 함수](/server/events)에서 자동으로 코루틴을 생성하지 않습니다. 코루틴을 사용할 필요가 없다면 성능 향상이 있을 수 있습니다. [Swoole 코루틴](/coroutine) 문서를 참조하세요.

  * **설정 방법**
    
    * `php.ini`에서 `swoole.enable_coroutine = 'Off'`로 설정 (자세한 내용은 [ini 설정 문서](/other/config.md) 참조)
    * `$server->set(['enable_coroutine' => false]);`는 `ini`보다 우선합니다.

  * **`enable_coroutine` 옵션 영향 범위**

      * onWorkerStart
      * onConnect
      * onOpen
      * onReceive
      * [setHandler](/redis_server?id=sethandler)
      * onPacket
      * onRequest
      * onMessage
      * onPipeMessage
      * onFinish
      * onClose
      * tick/after 타이머

!> `enable_coroutine`을 시작하면 위 콜백 함수에서 자동으로 코루틴이 생성됩니다.

* `enable_coroutine`을 `true`로 설정하면, 기본적으로 코루틴을 [onRequest](/http_server?id=on) 콜백에서 자동으로 생성하며, 개발자는 `go` 함수를 사용하여 직접 코루틴을 생성할 필요가 없습니다. 
* `enable_coroutine`이 `false`로 설정되어 있을 때, 기본적으로 코루틴이 자동으로 생성되지 않으며, 개발자가 코루틴을 사용하려면 직접 `go`를 사용하여 코루틴을 생성해야 합니다. 코루틴 기능을 사용하지 않으려면 `Swoole1.x`와 완전히 동일하게 작동합니다.
* 주의할 점은, 이 옵션을 활성화하면 Swoole이 코루틴을 통해 요청을 처리하게 됩니다. 이벤트에 블로킹 함수가 포함되어 있다면, 사전에 [일괄 코루틴 활성화](/runtime)를 시작해야 하며, `sleep` 또는 `mysqlnd`와 같은 블로킹 함수나 확장을 코루틴 활성화해야 합니다.

```php
$server = new Swoole\Http\Server("127.0.0.1", 9501);

$server->set([
    // 내장 코루틴 비활성화
    'enable_coroutine' => false,
]);

$server->on("request", function ($request, $response) {
    if ($request->server['request_uri'] == '/coro') {
        go(function () use ($response) {
            co::sleep(0.2);
            $response->header("Content-Type", "text/plain");
            $response->end("Hello World\n");
        });
    } else {
        $response->header("Content-Type", "text/plain");
        $response->end("Hello World\n");
    }
});

$server->start();
```
### send_yield

?> **데이터를 보낼 때 버퍼 메모리가 부족한 경우 현재 코루틴 내에서 직접 [yield](/coroutine?id=coroutine-scheduling)하여 데이터 전송이 완료될 때까지 기다리고, 버퍼가 비워지면 자동으로 현재 코루틴을 [resume](/coroutine?id=coroutine-scheduling)하여`send` 데이터를 계속합니다.**【기본값: [dispatch_mod](/server/setting?id=dispatch_mode) 2/4일 때 사용 가능하며 기본적으로 활성화됨】

* `Server/Client->send`가`false`를 반환하고 오류 코드가`SW_ERROR_OUTPUT_BUFFER_OVERFLOW`인 경우,`false`를`PHP` 레이어로 반환하지 않고 현재 코루틴을 [yield](/coroutine?id=coroutine-scheduling)하여 일시 중단시킵니다.
* `Server/Client`는 버퍼가 비워질 때까지의 이벤트를 감지하고, 이 이벤트가 트리거되면 버퍼 내의 데이터가 이미 전송되었으므로 해당하는 코루틴을 다시 [resume](/coroutine?id=coroutine-scheduling)합니다.
* 코루틴이 다시 실행되면`Server/Client->send`를 호출하여 버퍼에 데이터를 기록하고, 이때 버퍼가 비어 있으므로 전송은 반드시 성공합니다.

개선 전

```php
for ($i = 0; $i < 100; $i++) {
    // 버퍼가 가득차면 직접 false를 반환하고 출력 버퍼 오버플로우 오류가 발생합니다.
    $server->send($fd, $data_2m);
}
```

개선 후

```php
for ($i = 0; $i < 100; $i++) {
    // 버퍼가 가득차면 현재 코루틴을 yield하고 전송이 완료된 후에 resume하여 계속 실행합니다.
    $server->send($fd, $data_2m);
}
```

!> 이 기능은 기본 동작을 변경할 수 있으므로 수동으로 비활성화할 수 있습니다.

```php
$server->set([
    'send_yield' => false,
]);
```

* __영향 범위__

    * [Swoole\Server::send](/server/methods?id=send)
    * [Swoole\Http\Response::write](/http_server?id=write)
    * [Swoole\WebSocket\Server::push](/websocket_server?id=push)
    * [Swoole\Coroutine\Client::send](/coroutine_client/client?id=send)
    * [Swoole\Coroutine\Http\Client::push](/coroutine_client/http_client?id=push)
### send_timeout

`send_timeout`을 설정하여, `send_yield`와 함께 사용할 때, 지정된 시간 내에 데이터가 버퍼로 전송되지 못하면, 하위 레벨에서 `false`를 반환하고 오류 코드를 `ETIMEDOUT`으로 설정하며, [getLastError()](/server/methods?id=getlasterror) 메소드를 사용하여 오류 코드를 가져올 수 있습니다.

> 부동 소수점 형식, 단위는 초이며 최소 단위는 밀리초입니다.

```php
$server->set([
    'send_yield' => true,
    'send_timeout' => 1.5, // 1.5 seconds
]);

for ($i = 0; $i < 100; $i++) {
    if ($server->send($fd, $data_2m) === false and $server->getLastError() == SOCKET_ETIMEDOUT) {
      echo "보내기 시간 초과\n";
    }
}
```
### hook_flags

?> **`원 클릭 코루틴화` 훅 기능의 함수 범위를 설정합니다.**【기본값: 훅 안 함】

!> Swoole 버전이 `v4.5+` 이거나 [4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x)인 경우 사용할 수 있습니다. 자세한 내용은 [원 클릭 코루틴화](/runtime)를 참조하십시오.

```php
$server->set([
    'hook_flags' => SWOOLE_HOOK_SLEEP,
]);
```
### buffer_high_watermark

?> **버퍼 고수위선을 바이트 단위로 설정합니다.**

```php
$server->set([
    'buffer_high_watermark' => 8 * 1024 * 1024,
]);
```
### buffer_low_watermark

?> **버퍼 저 수위선을 바이트 단위로 설정합니다.**

```php
$server->set([
    'buffer_low_watermark' => 1 * 1024 * 1024,
]);
```
### tcp_user_timeout

?> TCP_USER_TIMEOUT 옵션은 TCP 소켓의 소켓 옵션으로, 데이터 패킷이 전송된 후 ACK 확인을 받지 못한 최대 시간을 밀리초 단위로 나타냅니다. 자세한 내용은 man 페이지를 참조하십시오.

```php
$server->set([
    'tcp_user_timeout' => 10 * 1000, // 10 seconds
]);
```

!> Swoole version >= `v4.5.3-alpha` required
### stats_file

?> **[stats()](/server/methods?id=stats) 내용이 작성되는 파일의 경로를 지정합니다. 설정하면 [onWorkerStart](/server/events?id=onworkerstart)시간에 타이머를 설정하여 주기적으로 [stats()](/server/methods?id=stats) 내용을 지정된 파일에 작성합니다.**

```php
$server->set([
    'stats_file' => __DIR__ . '/stats.log',
]);
```

!> Swoole 버전 >= `v4.5.5`에서 사용 가능
### event_object

?> **이 옵션을 설정하면 이벤트 콜백이 [객체 스타일](/server/events?id=callback-objects)로 사용됩니다.**【기본값：`false`】

```php
$server->set([
    'event_object' => true,
]);
```

!> Swoole 버전 >= `v4.6.0`에서 사용 가능합니다.
### start_session_id

?> **시작 세션 ID 설정**

```php
$server->set([
    'start_session_id' => 10,
]);
```

!> Swoole version >= `v4.6.0` available
### single_thread

?> **단일 스레드로 설정합니다.** 이 설정을 활성화하면 Reactor 스레드가 Master 프로세스의 Master 스레드와 병합되어 Master 스레드가 로직을 처리합니다. PHP ZTS에서는 `SWOOLE_PROCESS` 모드를 사용하는 경우 이 값을 꼭 `true`로 설정해야 합니다.

```php
$server->set([
    'single_thread' => true,
]);
```

!> Swoole 버전 >= `v4.2.13`에서 사용 가능합니다.
### max_queued_bytes

?> **수신 버퍼의 최대 대기열 길이를 설정합니다.** 초과하면 수신을 중지합니다.

```php
$server->set([
    'max_queued_bytes' => 1024 * 1024,
]);
```

!> Swoole 버전 >= `v4.5.0`에서 사용 가능
### admin_server

?> **[Swoole Dashboard](http://dashboard.swoole.com/)에서 서비스 정보를 확인하기 위해 admin_server 서비스를 설정합니다.**

```php
$server->set([
    'admin_server' => '0.0.0.0:9502',
]);
```

!> Swoole 버전 `v4.8.0` 이상에서 사용 가능합니다
