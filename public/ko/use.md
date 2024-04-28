제목: 사용 문제

이 글을 읽어주셔서 감사합니다. 사용 중에 문제가 발생한 경우 아래의 단계를 따르시기 바랍니다.

1. 문제가 발생한 환경을 설명해 주십시오.
2. 발생한 문제를 가능한 자세히 설명해 주십시오.
3. 만약 오류 메시지가 표시된다면, 해당 메시지를 제공해 주세요.

위 지침을 준수하시면 빠르게 문제를 해결할 수 있습니다. 감사합니다.
## Swoole 성능

> QPS 비교

Nginx 정적 페이지, Golang HTTP 프로그램, PHP7+Swoole HTTP 프로그램을 Apache-Bench(AB) 도구를 사용하여 스트레스 테스트합니다. 동일한 머신에서 동시에 100만 개의 HTTP 요청을 처리하는 벤치마크 테스트에서 QPS 비교는 다음과 같습니다:

| 소프트웨어 | QPS | 소프트웨어 버전 |
| --- | --- | --- |
| Nginx | 164489.92	| nginx/1.4.6 (Ubuntu) |
| Golang |	166838.68 |	go version go1.5.2 linux/amd64 |
| PHP7+Swoole |	287104.12 |	Swoole-1.7.22-alpha |
| Nginx-1.9.9 |	245058.70 |	nginx/1.9.9 |

!> 참고: Nginx-1.9.9의 테스트에서 access_log를 비활성화하고 open_file_cache를 사용하여 정적 파일을 메모리에 캐시했습니다.

> 테스트 환경

* CPU: Intel® Core™ i5-4590 CPU @ 3.30GHz × 4
* 메모리: 16G
* 디스크: 128G SSD
* 운영 체제: Ubuntu14.04 (Linux 3.16.0-55-generic)

> 스트레스 테스트 방법

```shell
ab -c 100 -n 1000000 -k http://127.0.0.1:8080/
```

> VHOST 구성

```nginx
server {
    listen 80 default_server;
    root /data/webroot;
    index index.html;
}
```

> 테스트 페이지

```html
<h1>Hello World!</h1>
```

> 프로세스 수

Nginx가 4개의 Worker 프로세스를 실행 중입니다.
```shell
htf@htf-All-Series:~/soft/php-7.0.0$ ps aux|grep nginx
root      1221  0.0  0.0  86300  3304 ?        Ss   12월07   0:00 nginx: master process /usr/sbin/nginx
www-data  1222  0.0  0.0  87316  5440 ?        S    12월07   0:44 nginx: worker process
www-data  1223  0.0  0.0  87184  5388 ?        S    12월07   0:36 nginx: worker process
www-data  1224  0.0  0.0  87000  5520 ?        S    12월07   0:40 nginx: worker process
www-data  1225  0.0  0.0  87524  5516 ?        S    12월07   0:45 nginx: worker process
```

> Golang

테스트 코드

```go
package main

import (
    "log"
    "net/http"
    "runtime"
)

func main() {
    runtime.GOMAXPROCS(runtime.NumCPU() - 1)

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Add("Last-Modified", "Thu, 18 Jun 2015 10:24:27 GMT")
        w.Header().Add("Accept-Ranges", "bytes")
        w.Header().Add("E-Tag", "55829c5b-17")
        w.Header().Add("Server", "golang-http-server")
        w.Write([]byte("<h1>\nHello world!\n</h1>\n"))
    })

    log.Printf("Go http Server listen on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

> PHP7+Swoole

PHP7은 `OPcache` 가속기를 활성화했습니다.

테스트 코드

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501, SWOOLE_BASE);

$http->set([
    'worker_num' => 4,
]);

$http->on('request', function ($request, Swoole\Http\Server $response) {
    $response->header('Last-Modified', 'Thu, 18 Jun 2015 10:24:27 GMT');
    $response->header('E-Tag', '55829c5b-17');
    $response->header('Accept-Ranges', 'bytes');    
    $response->end("<h1>\nHello Swoole.\n</h1>");
});

$http->start();
```

> **글로벌 웹 프레임워크 권위 성능 테스트 Techempower Web Framework Benchmarks**

최신 벤치마크 테스트 결과 링크: [techempower](https://www.techempower.com/benchmarks/#section=test&runid=9d5522a6-2917-467a-9d7a-8c0f6a8ed790)

Swoole은 **동적 언어 중 최고**

데이터베이스 IO 작업 테스트, 특별한 최적화 없이 기본 업무 코드 사용

**모든 정적 언어 프레임워크를 능가하는 성능(MySQL 사용, PostgreSQL 대신)**
## Swoole如何维持TCP长连接

관한 TCP 장접근 유지하는 데는 2세트의 설정 [tcp_keepalive] (/server/setting?id=open_tcp_keepalive) 와 [heartbeat] (/server/setting?id=heartbeat_check_interval) 가 있습니다. 사용법과 유의 사항은 [Swoole 공식 비디오 튜토리얼](https://course.swoole-cloud.com/course-video/10)을 참조하십시오.
## Swoole 서버를 올바르게 다시 시작하는 방법

코드를 수정한 후 PHP 코드가 적용되도록 서버를 다시 시작해야 하는 일이 자주 발생합니다. 서버가 요청을 계속 처리하는 바쁜 백엔드 서버의 경우, 관리자가 `kill` 프로세스를 사용하여 서버 프로그램을 종료/다시 시작하면 코드 실행이 중간에 중단될 수 있어 전체 비즈니스 로직의 완전성을 보장할 수 없습니다.

`swoole`은 유연한 종료/다시 시작 매커니즘을 제공하며, 관리자는 `Server`에 특정 신호를 보내거나 `reload` 메서드를 호출하기만 하면 작업 프로세스가 종료되고 다시 실행됩니다. 자세한 내용은 [reload()](/server/methods?id=reload)를 참조하십시오.

그러나 몇 가지 주의해야 할 사항이 있습니다:

먼저 수정한 코드는 반드시 `OnWorkerStart` 이벤트에서 다시로드해야만 적용됩니다. 예를 들어, 어떤 클래스가 `OnWorkerStart` 이전에 이미 composer의 autoload를 통해로드된 경우에는 작동하지 않습니다.

두 번째로, `reload`은 [max_wait_time](/server/setting?id=max_wait_time) 및 [reload_async](/server/setting?id=reload_async) 이 두 매개변수와 함께 사용해야 합니다. 이러한 매개변수를 설정하면 `비동기 안전 재시작`을 구현할 수 있습니다.

이 기능이 없으면 Worker 프로세스가 다시 시작 신호를 수신하거나 `[max_request](/server/setting?id=max_request)`에 도달하면 즉시 서비스가 중지되며, 이때 `Worker` 프로세스 내에 아직 이벤트 리스닝이 있다면 이러한 비동기 작업이 삭제될 수 있습니다. 위의 매개변수를 설정하면 먼저 새 `Worker`를 만들어서 이전 `Worker`가 모든 이벤트를 완료한 후 자체적으로 종료되며, 즉 `reload_async`가 발생합니다.

이전 `Worker`가 계속 종료되지 않는 경우에는 백그라운드에서 지정된 시간([max_wait_time](/server/setting?id=max_wait_time)초) 동안 기다리는 타이머가 추가되며, 이후에 이전 `Worker`가 종료되지 않으면 강제 종료되고 [WARNING](/question/use?id=forced-to-terminate) 오류가 발생합니다.

예시:

```php
<?php
$serv = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS);
$serv->set(array(
    'worker_num' => 1,
    'max_wait_time' => 60,
    'reload_async' => true,
));
$serv->on('receive', function (Swoole\Server $serv, $fd, $reactor_id, $data) {

    echo "[#" . $serv->worker_id . "]\tClient[$fd] receive data: $data\n";
    
    Swoole\Timer::tick(5000, function () {
        echo 'tick';
    });
});

$serv->start();
```

위의 코드 예제에서 `reload_async`가 없으면 `onReceive`에서 생성된 타이머가 손실되며, 타이머의 콜백 함수를 처리할 기회가 없습니다.
### 프로세스 종료 이벤트

비동기 재시작 기능을 지원하기 위해, 하위 레벨에서 [onWorkerExit](/server/events?id=onWorkerExit) 이벤트가 추가되었습니다. 이 이벤트는 이전 `Worker`가 종료하기 직전에 발생하며, `onWorkerExit` 이벤트가 트리거됩니다. 이 이벤트 콜백 함수 내에서 애플리케이션 레이어에서 일부 장기간 연결된 `소켓`을 정리할 수 있습니다. 이를 통해 이벤트 루프에서 fd가 없어지거나 [최대 대기 시간](/server/setting?id=max_wait_time)에 도달할 때까지 프로세스를 종료할 수 있습니다.

```php
$serv->on('WorkerExit', function (Swoole\Server $serv, $worker_id) {
    $redisState = $serv->redis->getState();
    if ($redisState == Swoole\Redis::STATE_READY or $redisState == Swoole\Redis::STATE_SUBSCRIBE)
    {
        $serv->redis->close();
    }
});
```

동시에 [Swoole Plus](https://www.swoole.com/swoole_plus)에서 파일 변경을 감지하는 기능이 추가되어 수동으로 다시로드하거나 신호를 보내지 않아도 파일 변경이 자동으로 worker를 다시 시작할 수 있습니다.
## send 완료 후 즉시 close 하는 것이 안전하지 않은 이유

send 작업 완료 후 즉시 close 하는 것은 안전하지 않습니다. 이것은 서버 측이나 클라이언트 측 모두에 해당합니다.

send 작업이 성공했다고 해도 데이터가 운영 체제의 소켓 버퍼에 성공적으로 기록되었음을 의미하며, 실제로 수신측이 데이터를 받았다는 것을 보장하지는 않습니다. 운영 체제가 성공적으로 전송했는지, 상대방 서버가 데이터를 받았는지, 서버 측 프로그램이 처리했는지는 정확하게 보장할 수 없습니다.

> close 이후의 로직은 아래의 linger 설정을 참조하세요.

이러한 로직은 전화 통화와 같은 원리입니다. A가 B에게 어떤 일을 말하고, A가 말을 마치면 전화를 끊는다고 해봅시다. 그렇다면 B가 들었는지는 A가 알 수 없습니다. 그러나 A가 일을 다 말하고, B가 "알겠어"라고 말한 뒤에 전화를 끊는다면 절대적으로 안전합니다.

linger 설정

socket이 close될 때, 버퍼에 아직 데이터가 남아 있다면, 운영 체제의 하부 레벨에서 `linger` 설정에 따라 처리 방법이 결정됩니다.

```c
struct linger
{
     int l_onoff;
     int l_linger;
};
```

* l_onoff = 0이면, close 시 즉시 반환되며, 미전송 데이터가 전송된 후 리소스가 해제되어 우아하게 종료됩니다.
* l_onoff != 0이고, l_linger = 0이면, close 시 즉시 반환되지만 미전송이 완료되지 않은 데이터는 보내지지 않고 RST 패킷을 통해 소켓 기술자를 강제로 닫습니다(강제 종료).
* l_onoff != 0이고, l_linger > 0이면, close 시 즉시 반환되지 않고, 커널은 l_linger의 값에 따라 잠시 지연됩니다. 타임아웃 시간 내에 미전송 데이터(ACK 패킷 포함)를 전송하고 상대편의 확인을 받으면 close가 올바르게 반환되어 소켓 기술자가 우아하게 종료됩니다. 그렇지 않으면 close는 직접 오류 값을 반환하여 미전송 데이터가 손실되며 소켓 기술자가 강제 종료됩니다. 소켓 기술자가 논블로킹으로 설정되어 있는 경우 close는 직접 반환합니다.
## 클라이언트가 이미 다른 코루틴에 바인딩되어 있습니다

`TCP` 연결에 대해 Swoole은 하나의 코루틴이 읽기 작업을 수행하고 다른 하나의 코루틴이 쓰기 작업을 수행할 수 있도록 허용합니다. 다시 말해 동시에 여러 코루틴이 한 TCP에 대해 읽기/쓰기 작업을 수행할 수 없으며, 이로 인해 다음과 같은 바인딩 오류가 발생할 수 있습니다:

```shell
Fatal error: Uncaught Swoole\Error: Socket#6 has already been bound to another coroutine#2, reading or writing of the same socket in coroutine#3 at the same time is not allowed 
```

재현 코드:

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function() {
    $cli = new Client('www.xinhuanet.com', 80);
    Coroutine::create(function () use ($cli) {
        $cli->get('/');
    });
    Coroutine::create(function () use ($cli) {
        $cli->get('/');
    });
});
```

문제 해결 방법 참고: https://wenda.swoole.com/detail/107474

!> 이 제한은 모든 다중 코루틴 환경에서 적용되며, 가장 흔한 경우는 [onReceive](/server/events?id=onreceive)와 같은 콜백 함수에서 동일한 TCP 연결을 공유하는 것입니다. 왜냐하면 이러한 유형의 콜백 함수는 자동으로 코루틴을 생성하기 때문입니다.
그러면 연결 풀이 필요한 경우는 어떻게 해야 할까요? `Swoole`은 내장된 [연결 풀](/coroutine/conn_pool)을 직접 사용하거나, 수동으로 `channel`로 연결 풀을 래핑할 수 있습니다.
## Call to undefined function Co\run()

大多数 문서 예제에서는 `Co\run()`을 사용하여 코루틴 컨테이너를 생성합니다. [코루틴 컨테이너란 무엇인가요?](/coroutine?id=什么是协程容器)

다음과 같은 오류가 발생한다면:

```bash
PHP Fatal error:  Uncaught Error: Call to undefined function Co\run()

PHP Fatal error:  Uncaught Error: Call to undefined function go()
```

`Swoole` 확장 기능 버전이 `v4.4.0` 미만인지 확인하거나 수동으로 [코루틴 단축 이름](/other/alias?id=协程短名称)이 비활성화된 경우 다음 해결 방법을 제공합니다.

* 버전이 낮으면 확장 기능 버전을 `>= v4.4.0`로 업그레이드하거나 코루틴을 만들기 위해 `Co\run` 대신 `go` 키워드를 사용하세요.
* 코루틴 단축 이름이 비활성화된 경우, [코루틴 단축 이름](/other/alias?id=协程短名称)을 다시 활성화하세요.
* `Coroutine::create` 메서드를 사용하여 `Co\run` 또는 `go` 대신 코루틴을 생성하세요.
* 전체 이름을 사용하세요: `Swoole\Coroutine\run`.
## Redis 또는 MySQL 연결을 공유할 수 있습니까?

절대로 불가능합니다. 각 프로세스는 개별적으로 `Redis`, `MySQL`, PDO 연결을 생성해야 합니다. 다른 저장소 클라이언트도 마찬가지입니다. 하나의 연결을 공유하면 반환된 결과가 어떤 프로세스에서 처리될지 보장할 수 없으며, 연결을 보유한 모든 프로세스 이론적으로 해당 연결을 읽거나 쓸 수 있기 때문에 데이터가 혼동될 수 있습니다.

**따라서 여러 프로세스 간에는 절대로 연결을 공유해서는 안 됩니다.**

* [Swoole\Server](/server/init)에서는 [onWorkerStart](/server/events?id=onworkerstart)에서 연결 객체를 생성해야 합니다.
* [Swoole\Process](/process/process)에서는 [Swoole\Process->start](/process/process?id=start) 이후에 자식 프로세스의 콜백 함수 내에서 연결 객체를 생성해야 합니다.
* 이 문제는 `pcntl_fork`를 사용하는 프로그램에도 동일하게 적용됩니다.

예시:

```php
$server = new Swoole\Server('0.0.0.0', 9502);

//workerstart 이벤트에서 반드시 redis/mysql 연결 생성
$server->on('workerstart', function($server, $id) {
    $redis = new Redis();
	$redis->connect('127.0.0.1', 6379);
	$server->redis = $redis;
});

$server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {	
	$value = $server->redis->get("key");
	$server->send($fd, "Swoole: ".$value);
});

$server->start();
```
## 연결이 종료된 문제

다음과 같은 메시지가 표시됩니다.

```bash
NOTICE swFactoryProcess_finish (ERRNO 1004): send 165 byte failed, because connection[fd=123] is closed

NOTICE swFactoryProcess_finish (ERROR 1005): connection[fd=123] does not exists
```

서버가 응답할 때 클라이언트가 이미 연결을 끊었기 때문에 발생합니다.

주로 다음과 같은 상황에서 발생합니다:

* 브라우저가 페이지를 미처 로드하기 전에 계속 새로고침하는 경우
* 중간에 ab 테스트를 취소하는 경우
* 시간 기반의 wrk 테스트 (시간이 다 되면 완료되지 않은 요청이 취소됨)

위와 같은 상황은 모두 정상적인 현상으로 간주되어 무시할 수 있습니다. 따라서이 오류의 수준은 NOTIC​E입니다.

다른 이유로 인해 무작위로 많은 연결이 끊기는 경우에만 주의해야 합니다.

```bash
WARNING swWorker_discard_data (ERRNO 1007): [2] received the wrong data[21 bytes] from socket#75

WARNING Worker_discard_data (ERRNO 1007): [2] ignore data[5 bytes] received from session#2
```

마찬가지로이 오류는 연결이 이미 닫혔음을 나타내며 받은 데이터는 버려집니다. [discard_timeout_request](/server/setting?id=discard_timeout_request)를 참조하세요.
## connected 속성과 연결 상태가 일치하지 않습니다

4.x 코루틴 버전부터 `connected` 속성은 더 이상 실시간으로 업데이트되지 않습니다. [isConnect](/client?id=isconnected) 메서드는 더 이상 신뢰할 수 없습니다
### 원인

코루틴의 목표는 동기 블로킹 프로그래밍 모델과 일치시키는 것입니다. 동기 블로킹 모델에서는 실시간으로 연결 상태를 업데이트하는 개념이 없습니다. PDO, curl 등에서 연결 개념이 없으며, IO 작업 중에 오류가 발생하거나 예외가 발생해야 연결이 끊어졌음을 발견할 수 있습니다.

Swoole의 하위 수준 공통 접근 방식은 IO 오류가 발생하면 false(또는 빈 내용)을 반환하고 클라이언트 객체에 해당하는 오류 코드와 오류 메시지를 설정하는 것입니다.
### 주의

이전의 비동기 버전은 "실시간"으로 `connected` 속성을 업데이트할 수 있지만, 실제로는 신뢰할 수 없습니다. 연결은 확인한 후 즉시 끊길 수 있습니다.
## "Connection refused"가 무엇을 의미하는가

telnet 127.0.0.1 9501 명령을 실행할 때 "Connection refused" 오류가 발생하면 이는 서버가 해당 포트를 수신 대기하고 있지 않음을 의미합니다.

* 프로그램 실행 여부를 확인하십시오: ps aux
* 포트가 수신 대기 중인지 확인하십시오: netstat -lp
* 네트워크 통신 과정을 확인하십시오: tcpdump traceroute
## 자원이 일시적으로 사용 불가능합니다 [11]

클라이언트의 swoole_client가 `recv` 중에 다음과 같은 오류를 발생시킵니다.

```shell
swoole_client::recv(): recv() failed. Error: Resource temporarily unavailable [11]
```

이 오류는 서버가 지정된 시간 내에 데이터를 반환하지 않아 수신 시간 초과가 발생했음을 나타냅니다.

* 네트워크 통신 과정을 확인하려면 tcpdump를 사용하여 서버가 데이터를 보냈는지 확인합니다.
* 서버의 `$serv->send` 함수가 true를 반환하는지 확인해야 합니다.
* 외부 네트워크 통신 시, 소요 시간이 길어질 경우 swoole_client의 타임아웃 시간을 높여야 합니다.
## worker exit timeout, forced to terminate :id=forced-to-terminate

다음과 같은 오류가 발견되었습니다:

```bash
WARNING swWorker_reactor_try_to_exit (ERRNO 9012): worker exit timeout, forced to terminate
```

이 오류는 설정된 시간 내 ([max_wait_time](/server/setting?id=max_wait_time) 초) Worker가 종료되지 않아 Swoole의 하위 프로세스가 강제로 종료된 것을 의미합니다.

다음 코드를 사용하여 재현할 수 있습니다:

```php
use Swoole\Timer;

$server = new Swoole\Server('127.0.0.1', 9501);
$server->set(
    [
        'reload_async' => true,
        'max_wait_time' => 4,
    ]
);

$server->on('workerStart', function (Swoole\Server $server, int $wid) {
    if ($wid === 0) {
        Timer::tick(5000, function () {
            echo 'tick';
        });
        Timer::after(500, function () use ($server) {
            $server->shutdown();
        });
    }
});

$server->on('receive', function () {

});

$server->start();
```
## 신호 Broken pipe에 대한 콜백 함수를 찾을 수 없음: 13

다음과 같은 오류가 발견되었습니다:

```bash
WARNING swSignalfd_onSignal (ERRNO 707): Unable to find callback function for signal Broken pipe: 13
```

이는 끊어진 연결로 데이터를 보내려고 시도했다는 것을 나타내며, 일반적으로는 전송 결과를 확인하지 않아 실패한 후에도 계속해서 전송을 시도하기 때문에 발생합니다.
## 스월을 학습하기 위해 알아야 할 기본 지식

### 다중 프로세스/다중 스레드

* Linux 운영 체제의 프로세스와 스레드 개념 이해
* Linux 프로세스/스레드 전환 스케줄링에 대한 기본 지식
* 파이프, Unix 소켓, 메시지 큐, 공유 메모리와 같은 기본 프로세스 간 통신에 대한 이해
### 소켓

- `accept/connect`, `send/recv`, `close`, `listen`, `bind`와 같은 기본 소켓 작업을 이해합니다.
- 소켓의 수신 버퍼, 송신 버퍼, 블로킹/논블로킹, 타임아웃 등의 개념을 이해합니다.
### IO 다중화

* `select`/`poll`/`epoll` 이해하기
* `select`/`epoll`을 기반으로 하는 이벤트 루프 및 `Reactor` 모델 이해하기
* 읽을 수 있는 이벤트, 쓸 수 있는 이벤트
### TCP/IP 네트워크 프로토콜

* `TCP/IP` 프로토콜 이해
* `TCP`, `UDP` 전송 프로토콜 이해
### 디버깅 도구

* [gdb](/other/tools?id=gdb)를 사용하여 `Linux` 프로그램을 디버깅합니다.
* [strace](/other/tools?id=strace)를 사용하여 프로세스의 시스템 호출을 추적합니다.
* [tcpdump](/other/tools?id=tcpdump)를 사용하여 네트워크 통신 과정을 추적합니다.
* ps, [lsof](/other/tools?id=lsof), top, vmstat, netstat, sar, ss 등과 같은 기타 `Linux` 시스템 도구들입니다.
## Swoole\Curl\Handler 클래스의 객체를 int로 변환할 수 없습니다

[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)를 사용할 때 다음과 같은 오류가 발생합니다:

```bash
PHP Notice:  Object of class Swoole\Curl\Handler could not be converted to int

PHP Warning: curl_multi_add_handle() expects parameter 2 to be resource, object given
```

이 문제는 hook 이후의 curl이 더 이상 resource 유형이 아니라 object 유형이기 때문에 int 유형으로 변환할 수 없다는 것입니다.

!> `int` 관련 문제는 SDK 제작자에게 코드 수정을 요청하는 것이 좋습니다. PHP8에서 curl은 더 이상 resource 유형이 아니라 object 유형으로 변경되었습니다.

해결 방법은 다음과 같습니다:

1. [SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)을 비활성화합니다. 그러나 [v4.5.4](/version/log?id=v454) 버전부터 [SWOOLE_HOOK_ALL](/runtime?id=swoole_hook_all)이 [SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)을 기본적으로 포함하므로 `SWOOLE_HOOK_ALL ^ SWOOLE_HOOK_CURL`로 [SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)을 비활성화할 수 있습니다.

2. Guzzle의 SDK를 사용하면 Handler를 교체하여 코루틴화를 구현할 수 있습니다.

3. Swoole `v4.6.0` 버전부터 [SWOOLE_HOOK_NATIVE_CURL](/runtime?id=swoole_hook_native_curl)을 사용하여 [SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)을 대체할 수 있습니다.
## 한 번에 코루틴 및 Guzzle 7.0+를 사용하여 요청을 보낸 후 결과를 터미널에 바로 출력하는 방법 :id=hook_guzzle

다음은 재현 코드입니다.

```php
// composer require guzzlehttp/guzzle
include __DIR__ . '/vendor/autoload.php';

use GuzzleHttp\Client;
use Swoole\Coroutine;

// v4.5.4 이전 버전
//Coroutine::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_CURL]);
Coroutine::set(['hook_flags' => SWOOLE_HOOK_ALL]);
Coroutine\run(function () {
    $client = new Client();
    $url = 'http://baidu.com';
    $res = $client->request('GET', $url);
    var_dump($res->getBody()->getContents());
});

// 요청 결과가 직접 출력되며, 출력되는 것이 아님
//<html>
//<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
//</html>
//string(0) ""
```

!> 이전 문제와 동일한 해결 방법이며, 이 문제는 Swoole 버전 >= `v4.5.8`에서 이미 수정되었습니다.
## 에러: 버퍼 공간 없음 [55]

이 오류를 무시할 수 있습니다. 이 오류는 [socket_buffer_size](/server/setting?id=socket_buffer_size) 옵션이 너무 크기 때문에 발생하는데, 특정 시스템에서는 받아들일 수 없지만 프로그램 실행에는 영향을 미치지 않습니다.
## GET/POST 요청의 최대 크기
### GET 요청의 최대 크기는 8192입니다

GET 요청은 하나의 Http 헤더만을 가지며, Swoole은 고정 크기의 8K 메모리 버퍼를 사용하고 수정할 수 없습니다. 올바른 Http 요청이 아닌 경우 오류가 발생합니다. 하위 수준에서는 다음과 같은 오류가 발생합니다:

```bash
WARN swReactorThread_onReceive_http_request: http header is too long.
```
### POST 파일 업로드

최대 크기는 [package_max_length](/server/setting?id=package_max_length) 설정 값에 따라 제한을 받습니다. 기본값은 2MB이며, [Server->set](/server/methods?id=set)을 호출하여 새 값으로 크기를 수정할 수 있습니다. Swoole은 전체 메모리 기반으로 작동하기 때문에 너무 크게 설정하면 많은 동시 요청으로 인해 서버 리소스가 고갈될 수 있습니다.

계산 방법: `최대 메모리 사용량` = `최대 동시 요청 수` * `package_max_length`
