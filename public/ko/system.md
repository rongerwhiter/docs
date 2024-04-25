# Coroutine\System

`API` 래퍼의 시스템 관련 코루틴. 이 모듈은 `v4.4.6` 이후의 공식 버전에서 사용할 수 있습니다. 대부분의 `API`는 `AIO` 스레드 풀을 기반으로 합니다.

!> 이전 버전인 `v4.4.6` 이전에는 `Co`이름 약칭이나 `Swoole\Coroutine`을 사용하십시오. 예를 들어, `Co::sleep` 또는 `Swoole\Coroutine::sleep`을 사용해야 합니다.  
`v4.4.6` 이후 버전부터는 `Co\System::sleep` 또는 `Swoole\Coroutine\System::sleep`을 **권장**합니다.  
이 변경은 네임스페이스를 표준화하고 동시에 하위 호환성을 보장하기 위함입니다(즉, `v4.4.6` 이전 버전의 방법도 여전히 유효합니다. 수정할 필요가 없습니다).
This is a code block, so I will leave it as it is.

번역: 이것은 코드 블록이므로 그대로 둡니다.
### statvfs()

파일 시스템 정보를 가져옵니다.

!> Swoole 버전 >= v4.2.5에서 사용 가능

```php
Swoole\Coroutine\System::statvfs(string $path): array|false
```

  * **Parameters** 

    * **`string $path`**
      * **Description**: 파일 시스템이 마운트된 디렉토리【예: `/`，df 및 `mount -l` 명령으로 확인 가능】
      * **Default**: 없음
      * **Other values**: 없음

  * **Usage example**

    ```php
    Swoole\Coroutine\run(function () {
        var_dump(Swoole\Coroutine\System::statvfs('/'));
    });
    
    // array(11) {
    //   ["bsize"]=>
    //   int(4096)
    //   ["frsize"]=>
    //   int(4096)
    //   ["blocks"]=>
    //   int(61068098)
    //   ["bfree"]=>
    //   int(45753580)
    //   ["bavail"]=>
    //   int(42645728)
    //   ["files"]=>
    //   int(15523840)
    //   ["ffree"]=>
    //   int(14909927)
    //   ["favail"]=>
    //   int(14909927)
    //   ["fsid"]=>
    //   int(1002377915335522995)
    //   ["flag"]=>
    //   int(4096)
    //   ["namemax"]=>
    //   int(255)
    // }
    ```
### fread()

파일을 읽는 코루틴 방식.

```php
Swoole\Coroutine\System::fread(resource $handle, int $length = 0): string|false
```

!> `v4.0.4` 미만 버전에서는 `fread` 메서드가 `STDIN`, `Socket`과 같은 비 파일 타입의 `stream`을 지원하지 않습니다. 이러한 리소스를 다루려면 `fread`를 사용하지 마십시오.  
`v4.0.4` 이상 버전에서는 `fread` 메서드가 비 파일 타입의 `stream` 리소스를 지원하며, 하위 레벨에서는 `stream` 유형에 따라 `AIO` 스레드 풀 또는 [EventLoop](/learn?id=EventLoop)을 자동으로 선택합니다.

  * **매개변수** 

    * **`resource $handle`**
      * **기능**：파일 핸들【`fopen`으로 열린 파일 타입의 `stream` 리소스여야 함】
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $length`**
      * **기능**：읽을 길이【기본값은 `0`으로, 파일의 전체 내용을 읽음】
      * **기본값**：`0`
      * **다른 값**：없음

  * **반환 값** 

    * 읽기 성공 시 문자열 내용 반환, 실패 시 `false` 반환

  * **사용 예시**  

    ```php
    $fp = fopen(__FILE__, "r");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fread($fp);
        var_dump($r);
    });
    ```
### fwrite()

파일에 데이터를 쓰는 코루틴 방식입니다.

```php
Swoole\Coroutine\System::fwrite(resource $handle, string $data, int $length = 0): int|false
```

!> `v4.0.4` 이하 버전에서 `fwrite` 메서드는 `STDIN`, `Socket`과 같은 파일이 아닌 `stream`을 지원하지 않습니다. 이러한 자원에 대해 `fwrite`을 사용하지 마십시오.  
`v4.0.4` 이상 버전에서 `fwrite` 메서드는 파일이 아닌 `stream` 리소스를 지원하며, 하위 수준에서 `stream` 유형을 자동으로 선택하여 `AIO` 스레드 풀이나 [EventLoop](/learn?id=이벤트루프)로 구현됩니다.

  * **매개변수** 

    * **`resource $handle`**
      * **기능**：파일 핸들【`fopen`으로 열린 파일 유형의 `stream` 리소스여야 함】
      * **기본값**：없음
      * **기타 값**：없음

    * **`string $data`**
      * **기능**：쓸 데이터 내용【텍스트 또는 이진 데이터일 수 있음】
      * **기본값**：없음
      * **기타 값**：없음

    * **`int $length`**
      * **기능**：읽을 길이【기본값은 `0`이며, `$data`의 전체 내용을 쓰기 위해 `$length`는 `$data`의 길이보다 작아야 함】
      * **기본값**：`0`
      * **기타 값**：없음

  * **반환 값** 

    * 성공적으로 쓰면 데이터 길이를 반환하고, 실패하면 `false`를 반환함

  * **사용 예제**  

    ```php
    $fp = fopen(__DIR__ . "/test.data", "a+");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fwrite($fp, "hello world\n", 5);
        var_dump($r);
    });
    ```
### fgets()

파일의 내용을 줄 단위로 읽는 코루틴 방식입니다.

기본적으로 `php_stream` 버퍼를 사용하며, 기본 크기는 `8192`바이트이며 `stream_set_chunk_size`를 사용하여 버퍼 크기를 설정할 수 있습니다.

```php
Swoole\Coroutine\System::fgets(resource $handle): string|false
```

!> `fgets` 함수는 파일 유형의 `stream` 자원에서만 사용할 수 있으며, Swoole 버전 >= `v4.4.4`에서 사용 가능합니다.

  * **매개변수** 

    * **`resource $handle`**
      * **기능**：파일 핸들【`fopen`으로 열린 파일 유형의 `stream` 자원이어야 합니다】
      * **기본값**：없음
      * **다른 값**：없음

  * **반환값** 

    * `EOL`(`\r` 또는 `\n`)을 읽으면 한 줄의 데이터와 `EOL`이 반환됩니다.
    * `EOL`을 읽지 못했지만 내용의 길이가 `php_stream` 버퍼 영역인 `8192`바이트를 초과하면 `8192`바이트의 데이터가 반환되며 `EOL`은 포함되지 않습니다.
    * 파일 끝에 도달하여 `EOF`일 때는 빈 문자열이 반환되며, 파일이 이미 읽혔는지 확인하기 위해 `feof`를 사용할 수 있습니다.
    * 읽기 실패 시 `false`가 반환되며, 에러 코드를 얻기 위해 [swoole_last_error](/functions?id=swoole_last_error) 함수를 사용할 수 있습니다.

  * **사용 예시**  

    ```php
    $fp = fopen(__DIR__ . "/defer_client.php", "r");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fgets($fp);
        var_dump($r);
    });
    ```  
### readFile()

파일을 코루틴 방식으로 읽습니다.

```php
Swoole\Coroutine\System::readFile(string $filename): string|false
```

  * **매개변수** 

    * **`string $filename`**
      * **기능**：파일 이름
      * **기본값**：없음
      * **다른 값**：없음

  * **반환 값** 

    * 성공적으로 읽을 경우 문자열 내용을 반환하고, 실패할 경우 `false`를 반환합니다. 에러 정보는 [swoole_last_error](/functions?id=swoole_last_error)를 사용하여 확인할 수 있습니다.
    * `readFile` 메서드에 크기 제한이 없으며, 내용은 메모리에 저장되므로 매우 큰 파일을 읽을 때 메모리를 과도하게 소모할 수 있습니다.

  * **사용 예시**  

    ```php
    $filename = __DIR__ . "/defer_client.php";
    Swoole\Coroutine\run(function () use ($filename)
    {
        $r = Swoole\Coroutine\System::readFile($filename);
        var_dump($r);
    });
    ```
### writeFile()

파일을 작성하는 코루틴 방식입니다.

```php
Swoole\Coroutine\System::writeFile(string $filename, string $fileContent, int $flags): bool
```

  * **매개변수** 

    * **`string $filename`**
      * **기능**：파일 이름【쓰기 권한이 있어야 하며, 파일이 없는 경우 자동으로 생성됩니다. 파일을 열 수 없으면 즉시 `false`를 반환합니다.】
      * **기본값**：없음
      * **다른 값**：없음

    * **`string $fileContent`**
      * **기능**：파일에 쓰여질 내용【최대 `4M`까지 쓸 수 있음】
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $flags`**
      * **기능**：쓰기 옵션【기본적으로 현재 파일 내용을 지우고, `FILE_APPEND`를 사용하여 파일 끝에 추가할 수 있음】
      * **기본값**：없음
      * **다른 값**：없음

  * **반환값** 

    * 성공적으로 작성되면 `true`가 반환됩니다.
    * 실패하면 `false`가 반환됩니다.

  * **사용 예시**  

    ```php
    $filename = __DIR__ . "/defer_client.php";
    Swoole\Coroutine\run(function () use ($filename)
    {
        $w = Swoole\Coroutine\System::writeFile($filename, "hello swoole!");
        var_dump($w);
    });
    ```
### sleep()

대기 상태로 들어갑니다.

`PHP`의 `sleep` 함수와 유사하지만, `Coroutine::sleep`은 [코루틴 스케줄러](/coroutine?id=코루틴-스케줄러)에서 구현되었으며, 내부적으로 현재 코루틴을 `yield`하고 실행 시간을 양보하여 비동기 타이머를 추가합니다. 타임아웃 시간이 경과하면 현재 코루틴을 다시 `resume`하여 실행을 계속합니다.

`sleep` 인터페이스를 사용하면 쉽게 시간 초과 대기 기능을 구현할 수 있습니다.

```php
Swoole\Coroutine\System::sleep(float $seconds): void
```

  * **매개변수** 

    * **`float $seconds`**
      * **기능**：잠자는 시간【0보다 커야 하며, 최대 1일(86400 초)을 초과해서는 안 됨】
      * **단위**：초, 최소 정확도는 밀리초(`0.001` 초)
      * **기본값**：없음
      * **다른 값**：없음

  * **사용 예**  

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9502);

    $server->on('Request', function($request, $response) {
        // 200ms 후에 브라우저에 응답을 보낸 후 기다립니다.
        Swoole\Coroutine\System::sleep(0.2);
        $response->end("<h1>Hello Swoole!</h1>");
    });

    $server->start();
    ```
### exec()

`shell` 명령을 실행합니다. 내부에서는 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)이 자동으로 이루어집니다.

```php
Swoole\Coroutine\System::exec(string $cmd): array
```

  * **매개변수** 

    * **`string $cmd`**
      * **기능**：실행할 `shell` 명령
      * **기본값**：없음
      * **다른 값**：없음

  * **반환 값**

    * 실행에 실패하면 `false`가 반환되며, 성공하면 프로세스 종료 상태 코드, 신호 및 출력 내용을 포함한 배열이 반환됩니다.

    ```php
    array(
        'code'   => 0,  // 프로세스 종료 상태 코드
        'signal' => 0,  // 신호
        'output' => '', // 출력 내용
    );
    ```

  * **사용 예**  

    ```php
    Swoole\Coroutine\run(function() {
        $ret = Swoole\Coroutine\System::exec("md5sum ".__FILE__);
    });
    ```

  * **주의**

  !>스크립트 명령을 실행하는 데 시간이 오래 걸리면 시간 초과로 종료되며, 이 경우 [socket_read_timeout](/coroutine_client/init?id=타임아웃-규칙)을 늘려 문제를 해결할 수 있습니다.
### gethostbyname()

IP로 도메인 이름을 해석합니다. 동기화된 스레드 풀을 사용하여 구현되며, 하위에서는 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링)이 자동으로 이루어집니다.

```php
Swoole\Coroutine\System::gethostbyname(string $domain, int $family = AF_INET, float $timeout = -1): string|false
```

  * **매개변수** 

    * **`string $domain`**
      * **기능**：도메인 이름
      * **기본값**：없음
      * **다른 값**：없음

    * **`int $family`**
      * **기능**：도메인 패밀리【`AF_INET`은 `IPv4` 주소를 반환하며, `AF_INET6`를 사용하면 `IPv6` 주소를 반환합니다】
      * **기본값**：`AF_INET`
      * **다른 값**：`AF_INET6`

    * **`float $timeout`**
      * **기능**：타임아웃 시간
      * **값 단위**：초, 최소 정확도는 밀리초로 (`0.001`초)
      * **기본값**：`-1`
      * **다른 값**：없음

  * **반환값**

    * 성공 시 도메인에 해당하는 `IP` 주소를 반환하고, 실패 시 `false`를 반환합니다. 에러 정보는 [swoole_last_error](/functions?id=swoole_last_error)를 사용하여 가져올 수 있습니다

    ```php
    array(
        'code'   => 0,  // 프로세스 종료 상태 코드
        'signal' => 0,  // 신호
        'output' => '', // 출력 내용
    );
    ```

  * **확장**

    * **타임아웃 제어**

      `timeout` 매개변수를 사용하여 코루틴이 대기하는 시간을 조정할 수 있습니다. 지정된 시간 내에 결과가 반환되지 않으면 코루틴은 즉시 `false`를 반환하고 계속해서 아래로 진행합니다. 하위 구현에서는 해당 비동기 작업을 `cancel`로 표시하고, `gethostbyname`은 여전히 `AIO` 스레드 풀에서 계속 실행됩니다.
      
      `/etc/resolv.conf` 파일을 수정하여 `gethostbyname` 및 `getaddrinfo`의 하위 `C` 함수의 타임아웃 시간을 설정할 수 있습니다. 자세한 내용은 [DNS 해석 시간 초과 및 재시도 설정](/learn_other?id=도메인-네임-서비스dns-해석-시간-초과-및-재시도-설정)을 참조하세요.

  * **사용 예시**  

    ```php
    Swoole\Coroutine\run(function () {
        $ip = Swoole\Coroutine\System::gethostbyname("www.baidu.com", AF_INET, 0.5);
        echo $ip;
    });
    ```
### getaddrinfo()

DNS 해석을 수행하여 도메인 이름에 대응하는 `IP` 주소를 조회합니다.

`gethostbyname`과 달리, `getaddrinfo`는 더 많은 매개변수 설정을 지원하며 여러 개의 `IP` 결과를 반환합니다.

```php
Swoole\Coroutine\System::getaddrinfo(string $domain, int $family = AF_INET, int $socktype = SOCK_STREAM, int $protocol = STREAM_IPPROTO_TCP, string $service = null, float $timeout = -1): array|false
```

  * **매개변수**

    * **`string $domain`**
      * **기능** : 도메인 이름
      * **기본값** : 없음
      * **기타 값** : 없음

    * **`int $family`**
      * **기능** : 계열【`AF_INET`은 `IPv4` 주소를 반환하고, `AF_INET6`를 사용하면 `IPv6` 주소를 반환합니다.】
      * **기본값** : 없음
      * **기타 값** : 없음
      
      !> 다른 매개변수 설정은 `man getaddrinfo` 문서를 참조하세요.

    * **`int $socktype`**
      * **기능** : 프로토콜 유형
      * **기본값** : `SOCK_STREAM`
      * **기타 값** : `SOCK_DGRAM`、`SOCK_RAW`

    * **`int $protocol`**
      * **기능** : 프로토콜
      * **기본값** : `STREAM_IPPROTO_TCP`
      * **기타 값** : `STREAM_IPPROTO_UDP`、`STREAM_IPPROTO_STCP`、`STREAM_IPPROTO_TIPC`、`0`

    * **`string $service`**
      * **기능** : 
      * **기본값** : 없음
      * **기타 값** : 없음

    * **`float $timeout`**
      * **기능** : 시간 초과
      * **값의 단위** : 초, 최소 정밀도는 밀리초(`0.001`초)
      * **기본값** : `-1`
      * **기타 값** : 없음

  * **반환 값**

    * 성공 시 여러 개의 `IP` 주소로 구성된 배열을 반환하고, 실패할 경우 `false`를 반환합니다.

  * **사용 예시**  

    ```php
    Swoole\Coroutine\run(function () {
        $ips = Swoole\Coroutine\System::getaddrinfo("www.baidu.com");
        var_dump($ips);
    });
    ```
### dnsLookup()

도메인 주소 조회.

`Coroutine\System::gethostbyname`과 달리, `Coroutine\System::dnsLookup`는 `UDP` 클라이언트 네트워크 통신을 직접 구현하며 `libc`의 `gethostbyname` 함수를 사용하지 않습니다.

!> Swoole 버전 >= `v4.4.3`에서 사용 가능하며, 기본적으로 `/etc/resolv.conf`를 읽어 `DNS` 서버 주소를 가져오며 현재는 `AF_INET(IPv4)` 도메인 해결만 지원합니다. Swoole 버전 >= `v4.7`에서는 세 번째 매개변수를 사용하여 `AF_INET6(IPv6)`를 지원할 수 있습니다.

```php
Swoole\Coroutine\System::dnsLookup(string $domain, float $timeout = 5, int $type = AF_INET): string|false
```

  * **매개변수** 

    * **`string $domain`**
      * **기능**：도메인
      * **기본값**：없음
      * **기타 값**：없음

    * **`float $timeout`**
      * **기능**：타임아웃 시간
      * **단위**：초, 최소 정밀도는 밀리초(0.001 초)
      * **기본값**：`5`
      * **기타 값**：없음

    * **`int $type`**
        * **단위**：초, 최소 정밀도는 밀리초(0.001 초)
        * **기본값**：`AF_INET`
        * **기타 값**：`AF_INET6`

    !> `$type` 매개변수는 Swoole 버전 >= `v4.7`에서 사용 가능합니다.

  * **반환 값**

    * 해결에 성공하면 해당 IP 주소가 반환됩니다.
    * 실패하면 `false`가 반환되며, 에러 정보는 [swoole_last_error](/functions?id=swoole_last_error)를 사용하여 확인할 수 있습니다.

  * **일반 오류**

    * `SWOOLE_ERROR_DNSLOOKUP_RESOLVE_FAILED`：해당 도메인을 해석할 수 없어 조회 실패
    * `SWOOLE_ERROR_DNSLOOKUP_RESOLVE_TIMEOUT`：해석 시간 초과, DNS 서버에 문제가 있을 수 있고, 지정된 시간 내에 결과를 반환할 수 없음

  * **사용 예**

    ```php
    Swoole\Coroutine\run(function () {
        $ip = Swoole\Coroutine\System::dnsLookup("www.baidu.com");
        echo $ip;
    });
    ```  
### wait()

원래의 [Process::wait](/process/process?id=wait)에 해당하는데, 이 API는 코루틴 버전으로, 코루틴을 일시 중단시키며, `Swoole\Process::wait` 및 `pcntl_wait` 함수를 대체할 수 있습니다.

!> Swoole 버전 >= `v4.5.0`에서 사용 가능합니다

```php
Swoole\Coroutine\System::wait(float $timeout = -1): array|false
```

* **매개변수** 

    * **`float $timeout`**
      * **기능**：시간 초과, 음수는 계속 기다림
      * **단위**：초, 최소 정밀도는 밀리초(`0.001`초)
      * **기본값**：`-1`
      * **기타 값**：없음

* **반환 값**

  * 성공하면 하위 프로세스의 `PID`, 종료 상태 코드, 어떤 시그널로 `KILL`되었는지를 포함하는 배열이 반환됨
  * 실패하면 `false`를 반환함

!> 각 하위 프로세스가 시작된 후, 부모 프로세스는 모두 코루틴을 사용하여 `wait()`(또는 `waitPid()`)를 호출하여 회수해야 합니다. 그렇지 않으면 하위 프로세스가 좀비 프로세스로 변환되어 운영 체제의 프로세스 리소스를 낭비하게 됩니다.  
코루틴을 사용하는 경우에는 프로세스를 먼저 생성한 다음, 프로세스 내에서 코루틴을 시작해야 합니다. 그렇지 않으면 코루틴을 fork한 경우에 복잡해지며, 하위 레벨에서 처리하기 어려워집니다.

* **예시**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    echo 'Hello Swoole';
});
$process->start();

Coroutine\run(function () use ($process) {
    $status = System::wait();
    assert($status['pid'] === $process->pid);
    var_dump($status);
});
```
### waitPid()

위의 wait 메서드와 기본적으로 동일하지만, 이 API는 특정 프로세스를 대기하도록 지정할 수 있습니다.

!> Swoole 버전 >= `v4.5.0`에서 사용 가능

```php
Swoole\Coroutine\System::waitPid(int $pid, float $timeout = -1): array|false
```

* **매개변수** 

    * **`int $pid`**
      * **기능**：프로세스 ID
      * **기본값**：`-1` (어떤 프로세스든지, 이 경우에는 wait 메서드와 동일)
      * **기타 값**：임의의 자연수

    * **`float $timeout`**
      * **기능**：타임아웃 시간, 음수는 무한대 대기를 의미
      * **단위**：초, 최소 정밀도는 밀리초(`0.001` 초)
      * **기본값**：`-1`
      * **기타 값**：없음

* **반환 값**

  * 작업이 성공하면 자식 프로세스의 `PID`와 종료 상태 코드, 어떤 시그널에 의해 종료되었는지를 포함하는 배열을 반환합니다.
  * 실패하면 `false`를 반환합니다.

!> 각 자식 프로세스가 시작된 후, 부모 프로세스는 꼭 한 번은 `wait()`(또는 `waitPid()`)를 호출하여 회수해야 합니다. 그렇지 않으면 자식 프로세스가 좀비 프로세스로 변하게 되어 운영 체제의 프로세스 리소스를 낭비하게 됩니다.

* **예시**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    echo 'Hello Swoole';
});
$process->start();

Coroutine\run(function () use ($process) {
    $status = System::waitPid($process->pid);
    var_dump($status);
});
```
### waitSignal()

신호를 청취하는 코루틴 버전의 리스너입니다. 신호가 발생할 때까지 현재 코루틴을 차단하며, `Swoole\Process::signal`과 `pcntl_signal` 함수를 대체할 수 있습니다.

!> Swoole 버전 >= `v4.5.0`에서 사용 가능

```php
Swoole\Coroutine\System::waitSignal(int $signo, float $timeout = -1): bool
```

  * **매개변수** 

    * **`int $signo`**
      * **기능**: 신호 유형
      * **기본값**: 없음
      * **다른 값**: SIG 시리즈 상수, 예: `SIGTERM`, `SIGKILL` 등

    * **`float $timeout`**
      * **기능**: 타임아웃 시간, 음수는 무한대기를 의미
      * **단위**: 초, 최소 정확도는 밀리초(`0.001`초)
      * **기본값**: `-1`
      * **다른 값**: 없음

  * **반환값**

    * 신호를 수신하면 `true` 반환
    * 시간 초과로 신호를 받지 못하면 `false` 반환

  * **예시**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    Coroutine\run(function () {
        $bool = System::waitSignal(SIGUSR1);
        var_dump($bool);
    });
});
$process->start();
sleep(1);
$process::kill($process->pid, SIGUSR1);
```
### waitEvent()

신호 대기기의 코루틴 버전은 해당 신호가 발생할 때까지 현재 코루틴을 차단합니다. IO 이벤트를 기다리며 `swoole_event` 관련 함수를 대체할 수 있습니다.

!> Swoole 버전 >= `v4.5`에서 사용 가능

```php
Swoole\Coroutine\System::waitEvent(mixed $socket, int $events = SWOOLE_EVENT_READ, float $timeout = -1): int | false
```

* **매개변수** 

    * **`mixed $socket`**
      * **기능**：파일 기술자(소켓 객체, 리소스 등과 같은 fd로 변환 가능한 모든 형식)
      * **기본 값**：없음
      * **다른 값**：없음

    * **`int $events`**
      * **기능**：이벤트 유형
      * **기본 값**：`SWOOLE_EVENT_READ`
      * **다른 값**：`SWOOLE_EVENT_WRITE` 또는 `SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE`

    * **`float $timeout`**
      * **기능**：타임아웃 시간, 음수는 타임아웃 없음을 의미
      * **값의 단위**：초, 최소 정밀도는 밀리초(0.001 초)
      * **기본 값**：`-1`
      * **다른 값**：없음

* **반환 값**

  * 이벤트 유형의 합을 반환(여러 비트가 될 수 있음), `$events` 매개변수와 관련이 있음
  * 실패 시 `false` 반환, [swoole_last_error](/functions?id=swoole_last_error)를 사용하여 오류 정보를 가져올 수 있음

* **예시**

> 동기 블로킹 코드를 이 API를 통해 코루틴 비블로킹으로 변환할 수 있음

```php
use Swoole\Coroutine;

Coroutine\run(function () {
    $client = stream_socket_client('tcp://www.qq.com:80', $errno, $errstr, 30);
    $events = Coroutine::waitEvent($client, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE);
    assert($events === SWOOLE_EVENT_WRITE);
    fwrite($client, "GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");
    $events = Coroutine::waitEvent($client, SWOOLE_EVENT_READ);
    assert($events === SWOOLE_EVENT_READ);
    $response = fread($client, 8192);
    echo $response;
});
```
