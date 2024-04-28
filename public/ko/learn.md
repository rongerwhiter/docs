# 기초 지식
## 콜백 함수를 설정하는 네 가지 방법

* **익명 함수**

```php
$server->on('Request', function ($req, $resp) use ($a, $b, $c) {
    echo "hello world";
});
```
!> `use`를 사용하여 익명 함수에 매개변수를 전달할 수 있습니다.

* **클래스 정적 메서드**

```php
class A
{
    static function test($req, $resp)
    {
        echo "hello world";
    }
}
$server->on('Request', 'A::Test');
$server->on('Request', array('A', 'Test'));
```
!> 해당 정적 메서드는 `public`이여야 합니다.

* **함수**

```php
function my_onRequest($req, $resp)
{
    echo "hello world";
}
$server->on('Request', 'my_onRequest');
```

* **객체 메서드**

```php
class A
{
    function test($req, $resp)
    {
        echo "hello world";
    }
}

$object = new A();
$server->on('Request', array($object, 'test'));
```

!> 해당 메서드는 `public`이여야 합니다.
## 동기I/O / 비동기I/O

`Swoole4+` 아래에서 모든 비즈니스 코드는 동기 방식으로 작성됩니다(`Swoole1.x` 시대에만 비동기 방식을 지원했지만, 현재는 비동기 클라이언트가 제거되었으며, 해당 요구 사항은 완전히 코루틴 클라이언트로 구현할 수 있습니다). 이는 마음의 부담이 전혀 없고, 인간의 사고 습관과 일치합니다. 그러나 동기 방식의 코드는 내부적으로 `동기I/O / 비동기I/O`의 차이가 있을 수 있습니다.

동기I/O든 비동기I/O든, `Swoole/Server`는 많은 `TCP` 클라이언트 연결을 유지할 수 있습니다([SWOOLE_PROCESS 모드](/learn?id=swoole_process) 참조). 서비스가 차단되는지 여부는 특정 매개변수를 설정할 필요가 없으며, 코드 내부에 동기I/O 작업이 있는지 여부에 따라 달라집니다.

**동기I/O란 무엇인가:**

간단한 예로는 `MySQL->query`를 실행하는 시점에서 해당 프로세스는 아무것도 하지 않고 MySQL의 결과를 기다립니다. 결과를 반환한 후에 코드를 계속 실행하므로 동기I/O 서비스의 동시성 능력은 매우 나쁩니다.

**어떤 코드가 동기I/O인가:**

* [일괄 코루틴화](/runtime)를 활성화하지 않으면 코드 내 대부분의 I/O 작업은 동기I/O입니다. 코루틴을 활성화하면 비동기 I/O로 변환되어 프로세스가 멍하있지 않고 계속 진행됩니다. [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링) 참조.
* 일부 I/O 작업은 일괄적으로 코루틴화할 수 없으며, 동기I/O를 비동기I/O로 변환할 수 없습니다. 예를 들어 `MongoDB`가 해당됩니다(`Swoole`이 이 문제를 해결할 것으로 믿습니다). 코드 작성 시 주의해야 합니다.

!> [코루틴](/coroutine)은 병렬성을 높이기 위한 것입니다. 내 응용 프로그램이 고성능을 필요로 하지 않거나 비동기화할 수 없는 작업(예: 앞서 언급된 MongoDB)이 반드시 필요하다면 [일괄 코루틴화](/runtime)를 비활성화할 수 있습니다. [enable_coroutine](/server/setting?id=enable_coroutine)를 비활성화하고 더 많은 `Worker` 프로세스를 실행하면 됩니다. 이는 `Fpm/Apache`와 동일한 모델이므로 언급할 만하게 `Swoole`이 [상주 프로세스](https://course.swoole-cloud.com/course-video/80) 임에도 불구하고 동기 I/O 성능이 크게 향상될 것입니다. 실제 응용 프로그램에서 이렇게 하는 회사들도 많이 있습니다.
### 동기 I/O를 비동기 I/O로 변환하기

[이전 섹션](/learn?id=동기io비동기io)에서 동기/비동기 I/O가 무엇인지에 대해 설명했습니다. `Swoole` 환경에서는 일부 동기 `I/O` 작업을 비동기 I/O로 전환할 수 있습니다.

- [원클릭 코루틴 활성화](/runtime)를 사용하면 `MySQL`, `Redis`, `Curl` 등의 작업이 비동기 I/O로 변환됩니다.
- [Event](/event) 모듈을 사용하여 이벤트를 수동으로 관리하고 fd를 [EventLoop](/learn?id=이벤트루프)에 추가하여 비동기 I/O로 변환할 수 있습니다. 예시:

```php
// inotify를 사용하여 파일 변경 감지
$fd = inotify_init();
// $fd를 Swoole의 EventLoop에 추가
Swoole\Event::add($fd, function () use ($fd){
    $var = inotify_read($fd); // 파일 변경 발생 시 변경된 파일 읽기.
    var_dump($var);
});
```

위 코드에서 `Swoole\Event::add`를 호출하지 않으면 I/O가 비동기화되지 않으며, 직접 `inotify_read()`를 호출하면 Worker 프로세스가 차단되어 다른 요청이 처리되지 않을 것입니다.

- `Swoole\Server`의 [sendMessage()](/server/methods?id=sendMessage) 메서드를 사용하여 프로세스 간 통신하는 경우, 기본적으로 `sendMessage`는 동기 I/O입니다. 그러나 `Swoole`은 일부 상황에서 이를 비동기 I/O로 전환합니다. [User 프로세스](/server/methods?id=addprocess)를 통한 예시:

```php
$serv = new Swoole\Server("0.0.0.0", 9501, SWOOLE_BASE);
$serv->set([
    'worker_num' => 1,
]);

$serv->on('pipeMessage', function ($serv, $src_worker_id, $data) {
    echo "#{$serv->worker_id} message from #$src_worker_id: $data\n";
    sleep(10); // sendMessage로부터 데이터를 수신하지 않으면 버퍼가 금방 가득 찰 것입니다.
});

$serv->on('receive', function (swoole_server $serv, $fd, $reactor_id, $data) {
    // 수신된 데이터를 처리하는 로직
});

// 경우 1: 동기 I/O(기본 동작)
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0)); // 기본적으로 버퍼가 가득 차면 여기서 차단됩니다.
    }
}, false);

// 경우 2: UserProcess 프로세스의 코루틴 지원을 enable_coroutine 매개변수로 활성화하면 다른 코루틴이 EventLoop 스케줄링을 받지 못하도록 하기 위해
// Swoole은 sendMessage를 비동기 I/O로 전환합니다.
$enable_coroutine = true;
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0)); // 버퍼가 가득 찼을 때 프로세스가 차단되지 않고 오류가 발생합니다.
}, false, 1, $enable_coroutine);

// 경우 3: UserProcess 프로세스에서 비동기 콜백을 설정하면(Event::add, 타이머 설정 등),
// 다른 콜백 함수가 EventLoop의 스케줄링을 받지 못하도록 하기 위해 Swoole은 sendMessage를 비동기 I/O로 전환합니다
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    swoole_timer_tick(2000, function ($interval) use ($worker, $serv) {
        echo "timer\n";
    });
    while (1) {
        var_dump($serv->sendMessage("big string", 0)); // 버퍼가 가득 찼을 때 프로세스가 차단되지 않고 오류가 발생합니다.
}, false);

$serv->addProcess($userProcess);

$serv->start();
```

- 마찬가지로 [Task 프로세스](/learn?id=taskworker프로세스)에서 `sendMessage()`를 사용하여 프로세스 간 통신하는 것도 동일하며, 차이점은 Task 프로세스에서는 Server의 [task_enable_coroutine](/server/setting?id=task_enable_coroutine) 설정으로 코루틴 지원을 활성화하며, `상황 3`은 존재하지 않습니다. 즉, Task 프로세스는 비동기 콜백을 설정했다고 해서 sendMessage를 비동기 I/O로 전환하지 않습니다.
## EventLoop이란 무엇인가요?

`EventLoop`란 이벤트 루프로, 간단히 말해 epoll_wait를 의미합니다. 이는 발생할 모든 이벤트 핸들(파일 디스크립터)을 epoll_wait에 추가하여 준비된 상태에 둔다는 것을 의미하며, 이러한 이벤트는 읽기 가능, 쓰기 가능, 오류 등을 포함합니다.

해당 프로세스는 epoll_wait라는 내부 커널 함수에서 블록됩니다. 이 함수는 이벤트가 발생하면(또는 시간 초과되면) 블록을 종료하고 결과를 반환하므로, 이에 따라 PHP 함수를 콜백할 수 있습니다. 예를 들어, 클라이언트가 데이터를 보내면 'onReceive' 콜백 함수가 호출됩니다.

epoll_wait에 많은 파일 디스크립터(fd)가 추가되고 동시에 많은 이벤트가 발생하면, epoll_wait 함수가 반환될 때 각 콜백 함수를 차례대로 호출하게 되며, 이것을 한 바퀴의 이벤트 루프라고 합니다. 이는 IO 다중화라고도 하며, 다시 epoll_wait를 호출하여 다음 순환의 이벤트 루프를 수행합니다.
## TCP 데이터 패킷 경계 문제

코드가[빠른 시작 중 TCP 서버](/start/start_tcp_server)는 병렬처리가 없는 경우에 잘 작동하지만 병행 처리를 많이하면 TCP 데이터 패킷 경계 문제가 발생합니다. `TCP` 프로토콜은 `UDP` 프로토콜의 순서 및 패킷 손실 재전송 문제를 해결했지만 `UDP`와 비교하여 새로운 문제를 가져왔습니다. `TCP` 프로토콜은 스트림 방식이기 때문에 데이터 패킷에 경계가 없어 응용 프로그램이 `TCP` 통신을 사용할 때 이러한 어려움에 직면할 수 있습니다. 이 문제는 일반적으로 TCP 스티키 패킷 문제로 알려져 있습니다.

`TCP` 통신은 스트림 방식이기 때문에 1개의 큰 데이터 패킷을 수신할 때 여러 개의 데이터 패킷으로 분할될 수 있습니다. 여러 번의 `Send` 호출은 하위에서 합쳐져 한 번에 전송될 수도 있습니다. 이 문제를 해결하기 위해 두 가지 조치가 필요합니다:

* 분할: `Server`가 여러 데이터 패킷을 수신하고 데이터 패킷을 분할해야 합니다.
* 합병: `Server`가 수신한 데이터는 패킷의 일부일 뿐이며 데이터를 캐싱하여 완전한 패킷으로 합병해야 합니다.

따라서 TCP 네트워크 통신 시 통신 프로토콜을 설정해야 합니다. 일반적으로 사용되는 TCP 일반 네트워크 통신 프로토콜에는 `HTTP`, `HTTPS`, `FTP`, `SMTP`, `POP3`, `IMAP`, `SSH`, `Redis`, `Memcache`, `MySQL` 등이 있습니다.

언급할 가치가 있는 것은, Swoole은 다양한 일반 프로토콜 파싱 기능을 내장하여 서버의 TCP 데이터 패킷 경계 문제를 해결합니다. 간단한 구성만으로도 이 문제를 해결할 수 있습니다. [open_http_protocol](/server/setting?id=open_http_protocol)/[open_http2_protocol](/http_server?id=open_http2_protocol)/[open_websocket_protocol](/server/setting?id=open_websocket_protocol)/[open_mqtt_protocol](/server/setting?id=open_mqtt_protocol)을 참조하세요.

일반 프로토콜 이외에도 사용자 정의 프로토콜을 설정할 수 있으며, `Swoole`은 `2` 종류의 사용자 정의 네트워크 통신 프로토콜을 지원합니다.

* **EOF 종료 기호 프로토콜**

`EOF` 프로토콜은 각 데이터 패킷의 끝에 특정 문자열을 추가하여 패킷의 끝을 나타냅니다. `Memcache`, `FTP`, `SMTP`는 모두 `\r\n`을 종료 기호로 사용합니다. 데이터를 전송할 때 패킷 끝에 `\r\n`을 추가하면 됩니다. `EOF` 프로토콜을 사용할 때는 데이터 패킷 중간에 `EOF`가 포함되지 않도록 주의해야 합니다.

`Server`와 `Client`의 코드에서는 `2`개의 매개변수만 설정하면 `EOF` 프로토콜을 사용할 수 있습니다.

```php
$server->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
```

그러나 위의 `EOF` 설정은 성능이 비교적 나쁠 수 있습니다. Swoole은 각 바이트를 순회하여 데이터가 `\r\n`인지 확인하기 때문입니다. 위의 방법 이외에도 다음과 같이 설정할 수 있습니다.

```php
$server->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
```
이 구성은 성능이 훨씬 우수하며 데이터를 반복 처리할 필요가 없지만 `분할` 문제만 해결하며 `합병` 문제는 해결하지 못합니다. 즉, `onReceive`에서 여러 개의 요청을 받을 수 있으며, 예를 들어 `explode("\r\n", $data)`와 같은 직접 패킷 분할을 해야합니다. 이 구성은 요청-응답형 서비스(예: 터미널에서 명령 입력)의 경우에 특히 유용합니다. 클라이언트는 한 번의 요청을 보낸 후 서버로 현재 요청의 응답 데이터를 반환할 때까지 기다려야하므로 2개의 요청을 동시에 보내지 않습니다.

* **고정 패킷 헤더 + 패킷 바디 프로토콜**

고정 패킷 헤더 방법은 서버 프로그램에서 자주 볼 수 있습니다. 이러한 프로토콜의 특징은 데이터 패킷이 항상 헤더+바디 `2` 부분으로 구성된다는 것입니다. 헤더는 패킷 바디 또는 전체 패킷의 길이를 지정하는 필드로 구성됩니다. 길이는 보통 `2`바이트/`4`바이트 정수로 표시됩니다. 서버는 헤더를 받은 후 길이 값에 따라 얼마나 더 많은 데이터를 수신하는지 정확히 제어할 수 있습니다. `Swoole`의 설정은 이러한 프로토콜을 효과적으로 지원하며 `4`개의 매개변수를 유연하게 설정하여 모든 상황에 대응할 수 있습니다.

`Server`는 [onReceive](/server/events?id=onreceive) 콜백 함수에서 데이터 패킷을 처리하며, 프로토콜 처리가 설정된 경우에만 완전한 데이터 패킷을 수신할 때에만 [onReceive](/server/events?id=onreceive) 이벤트가 발생합니다. 클라이언트는 프로토콜 처리가 설정된 후 [$client->recv()](/client?id=recv)를 호출할 때 길이를 더 이상 전달할 필요가 없습니다. `recv` 함수는 완전한 데이터 패킷을 수신하거나 오류가 발생한 후 반환됩니다.

```php
$server->set(array(
    'open_length_check' => true,
    'package_max_length' => 81920,
    'package_length_type' => 'n', //see php pack()
    'package_length_offset' => 0,
    'package_body_offset' => 2,
));
```

!> 각 구성의 구체적인 의미는`서버/클라이언트`장의 [설정](/server/setting?id=open_length_check)절을 참조하십시오.
## IPC이란 무엇인가요?

한 대의 호스트에서 두 개의 프로세스 간 통신(이하 IPC)에는 여러 가지 방법이 있습니다. Swoole에서는 `Unix Socket`과 `sysvmsg` 두 가지 방법을 사용했습니다. 아래에서 각각을 소개하겠습니다:

- **Unix Socket**
  
    전체 이름은 UNIX 도메인 소켓(UNIX Domain Socket), 줄여서 `UDS`라고 합니다. 이는 소켓 API(socket, bind, listen, connect, read, write, close 등)를 사용하며, TCP/IP와 다르게 IP와 포트를 지정할 필요 없이 파일 이름을 통해 나타냅니다(예: `/tmp/php-fcgi.sock` 등). UDS는 Linux 커널에서 구현된 메모리 내 통신으로, 어떠한 `IO` 비용도 없습니다. `1`개의 프로세스에서 `write`하고 `1`개의 프로세스에서 `read`할 때, 매번 `1024`바이트의 데이터를 읽고 쓰는 테스트에서, `100`만 번의 통신은 단 `1.02`초만 소요되며, 기능적으로 매우 강력합니다. `Swoole`에서 기본적으로 이 IPC 방식을 사용합니다.  
      
    * **`SOCK_STREAM`과 `SOCK_DGRAM`**

        - `Swoole`에서 `UDS` 통신을 할 때 두 가지 유형을 사용할 수 있습니다: `SOCK_STREAM`과 `SOCK_DGRAM`. 이는 TCP와 UDP의 차이로 간단히 이해할 수 있으며, `SOCK_STREAM` 유형을 사용할 때에는 마찬가지로 [TCP 데이터 패킷 경계 문제](/learn?id=tcp데이터-패킷-경계-문제)를 고려해야 합니다.
        - `SOCK_DGRAM` 유형을 사용할 때에는 TCP 데이터 패킷 경계 문제를 고려할 필요가 없으며, 각 `send()`된 데이터는 경계를 가지고 있어, 보낸 데이터만큼 수신되며, 전송 중 소실이나 순서가 바뀌는 문제가 없습니다. `send`로 쓴 데이터는 `recv`로 완전히 일치하는 순서로 받습니다. `send`가 성공하면 반드시 `recv`할 수 있습니다.

    데이터가 작을 때는 `SOCK_DGRAM` 방식이 매우 적합합니다. **각 IP 패킷의 크기가 최대 64k로 제한되므로, IPC에 `SOCK_DGRAM`을 사용할 때 한 번에 보내는 데이터는 64k를 초과할 수 없으며, 패킷 수신 속도가 너무 느려서 운영 체제 버퍼가 가득 차면 패킷을 버립니다. UDP는 패킷 손실을 허용하므로 버퍼를 적절히 크게 조정해야 합니다**.

- **sysvmsg**
     
    이는 Linux에서 제공하는 `메시지 큐`를 의미합니다. 이 IPC 방식은 통신을 위해 파일 이름을 `키`로 사용하며, 매우 유연하지 않습니다. 실제 프로젝트에서는 잘 사용되지 않으므로 자세한 설명은 생략하겠습니다.

    * **이 IPC 방식은 두 가지 시나리오에서 유용합니다:**

        - 데이터 손실 방지, 전체 서비스가 다운되어도 큐에 남아 있는 메시지를 다시 소비할 수 있습니다. **하지만 더러운 데이터 문제도 있습니다**.
        - 외부에서 데이터 전달 가능하며, 예를 들어 Swoole의 `Worker 프로세스`가 메시지 큐를 통해 `Task 프로세스`에 작업을 전달하거나, 제3자 프로세스가 작업을 큐에 추가하여 Task가 처리하거나, 명령 줄에서 수동으로 메시지를 큐에 추가할 수 있습니다.
## Master 프로세스, Reactor 스레드, Worker 프로세스, Task 프로세스, Manager 프로세스의 차이점과 관련성 :id=diff-process
### Master 프로세스

* Master 프로세스는 멀티 스레드 프로세스이며, [프로세스/스레드 구조 다이어그램](/server/init?id=프로세스스레드구조다이어그램)을 참조하십시오.
### Reactor 스레드

* Reactor 스레드는 마스터 프로세스에서 생성되는 스레드입니다.
* 클라이언트 `TCP` 연결을 유지하고, 네트워크 `IO`를 처리하며, 프로토콜을 처리하고, 데이터를 송수신합니다.
* 어떠한 PHP 코드도 실행하지 않습니다.
* `TCP` 클라이언트로부터 수신된 데이터를 버퍼링하고, 조립하여 완전한 요청 데이터 패킷으로 분할합니다.
### Worker 프로세스

* `Reactor` 스레드가 전달한 요청 데이터 패킷을 수신하고 `PHP` 콜백 함수를 실행하여 데이터를 처리합니다.
* 응답 데이터를 생성하고 `Reactor` 스레드에 전송하여 `TCP` 클라이언트에 전달합니다.
* 비동기 논블로킹 모드 또는 동기 블로킹 모드로 설정할 수 있습니다.
* `Worker`는 여러 프로세스로 실행될 수 있습니다.
### TaskWorker 프로세스

* `Worker` 프로세스로부터 Swoole\Server->task/taskwait/taskCo/taskWaitMulti 메소드를 통해 전달된 작업을 수락합니다.
* 작업을 처리하고 결과 데이터를 `Worker` 프로세스에 반환합니다 (Swoole\Server->finish를 사용).
* 완전히 **동기 블로킹** 모드입니다.
* `TaskWorker`는 다중 프로세스로 실행되며, [task 전체 예제](/start/start_task)
### Manager 프로세스

* 'Worker'/'Task' 프로세스를 생성/회수하는 책임이 있습니다.

그들 간의 관계는 'Reactor'가 'nginx'이고, 'Worker'가 'PHP-FPM'이라고 이해할 수 있습니다. 'Reactor' 스레드는 네트워크 요청을 비동기적으로 병렬로 처리한 다음 'Worker' 프로세스로 전달하여 처리합니다. 'Reactor'와 'Worker' 간에는 [unixSocket](/learn?id=IPC-는-무엇인가)을 통해 통신합니다.

'PHP-FPM' 응용 프로그램에서는 작업을 비동기적으로 'Redis' 등의 대기열에 전송하고 일부 'PHP' 프로세스를 백그라운드에서 시작하여 이러한 작업을 비동기적으로 처리하는 것이 자주 있습니다. 'Swoole'이 제공하는 'TaskWorker'는 작업 전달, 대기열, 'PHP' 작업 처리 프로세스 관리를 하나로 통합한 더 완전한 솔루션입니다. 기본 제공되는 API를 통해 간단하게 비동기 작업을 처리할 수 있습니다. 또한 'TaskWorker'는 작업 완료 후 결과를 'Worker'로 반환할 수도 있습니다.

'Swoole'의 'Reactor', 'Worker', 'TaskWorker'는 밀접하게 결합되어 더 고급스러운 사용 방식을 제공합니다.

더 일반적인 비유를 들자면, 'Server'가 공장이라면 'Reactor'는 영업부이며, 고객 주문을 받습니다. 그리고 'Worker'는 노동자이며, 영업이 주문을 받으면 고객이 원하는 것을 생산하기 위해 일합니다. 'TaskWorker'는 행정 직원으로 이해할 수 있으며, 'Worker'가 주로 일할 수 있도록 보조 작업을 수행합니다.

다음과 같이:

![process_demo](_images/server/process_demo.png)
## Server의 2가지 실행 모드 소개

`Swoole\Server` 생성자의 세 번째 매개변수에는 [SWOOLE_BASE](/learn?id=swoole_base) 또는 [SWOOLE_PROCESS](/learn?id=swoole_process) 라는 2개의 상수 값을 넣을 수 있습니다. 아래에서 각 모드의 차이점과 장단점에 대해 설명하겠습니다.
### SWOOLE_PROCESS

`Server`에서의 SWOOLE_PROCESS 모드는 모든 클라이언트 TCP 연결이 [메인 프로세스](/learn?id=reactor线)와 수립되며 내부 구현이 복잡하며 많은 프로세스 간 통신 및 관리 메커니즘이 사용됩니다. 매우 복잡한 비즈니스 로직에 적합합니다. `Swoole`은 완벽한 프로세스 관리 및 메모리 보호 메커니즘을 제공합니다.
특히 매우 복잡한 비즈니스 로직의 경우 장기간 안정적으로 작동할 수 있습니다.

`Swoole`은 [Reactor](/learn?id=reactor线) 스레드에서 `Buffer` 기능을 제공하여 대량의 느린 연결 및 바이트 단위 악의적인 클라이언트를 처리할 수 있습니다.
#### 프로세스 모델의 이점:

* 연결 및 데이터 요청 전송은 분리되어 있어, 어떤 연결이 데이터 양이 많거나 적다해도 `Worker` 프로세스가 불균형하게 발생하지 않습니다.
* `Worker` 프로세스에서 치명적인 오류가 발생해도 연결이 끊기지 않습니다.
* 단일 연결 동시성을 구현할 수 있어, 소수의 `TCP` 연결만 유지하면 요청을 여러 `Worker` 프로세스에서 동시에 처리할 수 있습니다.
#### 프로세스 모드의 단점:

* `2` 번의 `IPC` 오버헤드가 발생하며, `master` 프로세스와 `worker` 프로세스는 [unixSocket](/learn?id=IPC란 무엇인가요)을 사용하여 통신해야 합니다.
* `SWOOLE_PROCESS`는 PHP ZTS를 지원하지 않으며, 이 경우에는 `SWOOLE_BASE`만 사용하거나 [single_thread](/server/setting?id=single_thread)를 true로 설정해야 합니다.
### SWOOLE_BASE

SWOOLE_BASE 모드는 전통적인 비동기 논블로킹 `Server`입니다. `Nginx`와 `Node.js`와 같은 프로그램과 완전히 동일합니다.

[worker_num](/server/setting?id=worker_num) 매개변수는 `BASE` 모드에서 여전히 유효하며, 여러 개의 `Worker` 프로세스가 시작될 것입니다.

TCP 연결 요청이 들어올 때, 모든 Worker 프로세스가 이 연결을 경쟁하게 되고, 마침내 한 Worker 프로세스가 직접적으로 클라이언트와 TCP 연결을 수립하게 됩니다. 이후 이 연결의 모든 데이터 송수신은 직접 이 Worker와 통신하며, 주 프로세스의 Reactor 스레드를 거치지 않습니다.

* `BASE` 모드에서는 `Master` 프로세스의 역할이 없으며, 오직 [Manager](/learn?id=manager프로세스) 프로세스의 역할만 있습니다.
* 각 `Worker` 프로세스는 [SWOOLE_PROCESS](/learn?id=swoole_process) 모드 및 `Worker` 프로세스의 두 가지 책임을 모두 담당합니다.
* `BASE` 모드에서 `Manager` 프로세스는 선택사항입니다. `worker_num=1`로 설정되어 있고 `Task` 및 `MaxRequest` 기능을 사용하지 않은 경우, 하위 수준에서는 직접 하나의 단일 `Worker` 프로세스가 생성되며 `Manager` 프로세스는 생성되지 않습니다.
#### BASE 모드의 장점:

* `BASE` 모드는 `IPC` 오버헤드가 없어 성능이 더 좋습니다.
* `BASE` 모드의 코드는 더 간단하고 실수를 줄일 수 있습니다.
#### BASE 모드의 단점:

* `TCP` 연결은 `Worker` 프로세스에서 유지되므로 특정 `Worker` 프로세스가 종료되면 해당 `Worker` 내의 모든 연결이 닫힙니다.
* 소수의 `TCP` 장기 연결은 모든 `Worker` 프로세스를 활용할 수 없습니다.
* `TCP` 연결은 `Worker`에 바인딩되어 있으며, 장기 연결 응용 프로그램에서 일부 연결의 데이터 양이 많아지면 해당 연결이 속한 `Worker` 프로세스의 부하가 매우 높아집니다. 그러나 일부 연결의 데이터 양이 적기 때문에 해당 연결을 포함하는 `Worker` 프로세스의 부하가 매우 낮아지고, 서로 다른 `Worker` 프로세스 간에 균형을 유지할 수 없습니다.
* 콜백 함수에 차단되는 작업이 있는 경우 `Server`가 동기 모드로 변하는 문제가 발생하여 이로 인해 `TCP`의 [backlog](/server/setting?id=backlog) 대기열이 가득 찰 수 있습니다.
#### BASE 모드의 적합한 시나리오:

만약 클라이언트 간 상호 작용이 필요하지 않다면 `BASE` 모드를 사용할 수 있습니다. 예를 들어 `Memcache`, `HTTP` 서버 등이 있습니다.
#### BASE 모드 제한:

`BASE` 모드에서는 [Server 메소드](/server/methods) 중 [send](/server/methods?id=send) 및 [close](/server/methods?id=close)를 제외한 다른 메소드는 모두 **프로세스 간 실행을 지원하지 않습니다**.

!> v4.5.x 버전에서 `BASE` 모드에서 `send` 메소드만이 프로세스 간 실행을 지원합니다; v4.6.x 버전에서는 `send`와 `close` 메소드만이 지원됩니다.
## Process、Process\Pool、UserProcess의 차이는 무엇인가요? :id=process-diff
### 프로세스

[프로세스](/process/process)는 Swoole에서 제공하는 프로세스 관리 모듈로서 PHP의 `pcntl`을 대체하는 데 사용됩니다.

- 프로세스 간 통신을 쉽게 구현할 수 있습니다.
- 표준 입력 및 출력을 재지정할 수 있어, 자식 프로세스에서 `echo`를 사용해도 화면에 출력되지 않고 파이프로 데이터를 쓸 수 있습니다. 키보드 입력은 파이프로 리디렉션하여 데이터를 읽을 수 있습니다.
- [exec](/process/process?id=exec) 인터페이스를 제공하여, 생성된 프로세스가 다른 프로그램을 실행하고, 원래의 `PHP` 부모 프로세스와 쉽게 통신할 수 있습니다.

!> 코루틴 환경에서는 `Process` 모듈을 사용할 수 없으므로, `runtime hook` + `proc_open`을 사용하여 구현할 수 있습니다. 자세한 내용은 [코루틴 프로세스 관리](/coroutine/proc_open)를 참조하십시오.
### 프로세스\풀

[프로세스\풀](/process/process_pool)은 서버의 프로세스 관리 모듈을 PHP 클래스로 래핑하여 PHP 코드에서 Swoole의 프로세스 관리자를 사용할 수 있도록 지원합니다.

실제 프로젝트에서는 `Redis`, `Kafka`, `RabbitMQ`을 기반으로 한 다중 프로세스 대기열 소비자, 다중 프로세스 크롤러 등과 같이 오랜 시간 실행되는 스크립트를 작성해야 하는 경우가 많습니다. 개발자는 다중 프로세스 프로그래밍을 구현하기 위해 `pcntl` 및 `posix` 관련 확장 라이브러리를 사용해야 하지만 동시에 깊은 `Linux` 시스템 프로그래밍 경험을 필요로 합니다. 그렇지 않으면 문제가 발생할 수 있습니다. Swoole이 제공하는 프로세스 관리자를 사용하면 다중 프로세스 스크립트 프로그래밍 작업을 크게 단순화할 수 있습니다.

- 작업 프로세스의 안정성 보장
- 시그널 처리 지원
- 메시지 대기열 및 `TCP-Socket` 메시지 전송 기능 지원
### UserProcess

`UserProcess`는 [addProcess](/server/methods?id=addprocess)를 사용하여 추가된 사용자 지정 작업 프로세스로, 일반적으로 특별한 작업을 수행하기 위해 만들어진 특수한 작업 프로세스를 만드는 데 사용됩니다.

`UserProcess`는 [Manager 프로세스](/learn?id=manager프로세스)에 호스팅되지만 [Worker 프로세스](/learn?id=worker프로세스)와 비교했을 때 상대적으로 독립적인 프로세스로, 사용자 지정 기능을 실행하는 데 사용됩니다.
