# 런타임

'Swoole 1.x' 대비 'Swoole 4+'는 코루틴이라는 강력한 도구를 제공합니다. 모든 비즈니스 코드는 동기적이지만 하위 수준의 I/O는 비동기입니다. 이는 전통적인 비동기 콜백으로 인해 발생하는 코드 로직의 분산 및 다층 콜백으로 인한 유지보수 어려움을 피하면서 동시성을 보장합니다. 이 효과를 얻으려면 모든 'IO' 요청이 [비동기 IO](/learn?id=동기io비동기io)여야 하며, 'Swoole 1.x' 시대에 제공된 'MySQL', 'Redis' 등의 클라이언트는 비동기 IO이지만 코루틴 방식이 아니기 때문에 'Swoole 4' 시대에 이러한 클라이언트가 제거되었습니다.

이러한 클라이언트의 코루틴 지원 문제를 해결하기 위해 Swoole 개발팀은 많은 노력을 기울였습니다:

- 처음에는 각 유형의 클라이언트에 대해 코루틴 클라이언트를 만들었지만, 이렇게하면 다음과 같은 3가지 문제가 발생했습니다:

  * 구현이 복잡하며 각 클라이언트의 세부 프로토콜이 매우 복잡하여 모든 것을 완벽하게 지원하기 어렵습니다.
  * 사용자가 변경해야 하는 코드가 많습니다. 예를 들어 이전에는 PHP의 기본 'PDO'를 사용하여 'MySQL'를 쿼리했는데, 이제는 [Swoole\Coroutine\MySQL](/coroutine_client/mysql)을 사용해야 합니다.
  * 모든 작업을 다루기가 어렵습니다. 'proc_open()', 'sleep()' 함수 등도 블로킹될 수 있어 프로그램이 동기 블로킹으로 변할 수 있습니다.

- 위 문제에 대처하기 위해 Swoole 개발팀은 접근 방식을 변경하여 `Hook` 네이티브 PHP 함수를 사용하여 코루틴 클라이언트를 구현했습니다. 한 줄의 코드로 기존의 동기 IO 코드를 [코루틴 스케줄링](/coroutine?id=코루틴-스케줄링) 및 [비동기 IO](/learn?id=동기io비동기io)로 변환할 수 있습니다. 즉, `원 키로 코루틴화`.

!> 이 기능은 'v4.3' 버전 이후 안정화되었으며 코루틴화할 수 있는 함수도 계속 늘어나고 있습니다. 따라서 이전에 작성된 일부 코루틴 클라이언트는 더 이상 권장되지 않습니다. 자세한 내용은 [코루틴 클라이언트](/coroutine_client/init)를 참조하십시오. 예를 들어, 'v4.3+'에서는 파일 작업(`file_get_contents`, `fread` 등)의 코루틴화가 지원되었으며 'v4.3+' 버전을 사용 중이라면 Swoole이 제공하는 [코루틴 파일 조작](/coroutine/system) 대신 바로 코루틴화를 사용할 수 있습니다.
## 함수 프로토 타입

`flags`를 사용하여 `코루틴`으로 만들 함수의 범위를 설정합니다.

```php
Co::set(['hook_flags'=> SWOOLE_HOOK_ALL]); // v4.4+ 버전에서는 이 방법을 사용합니다.
// 또는
Swoole\Runtime::enableCoroutine($flags = SWOOLE_HOOK_ALL);
```

동시에 여러 `flags`를 활성화하려면 `|` 연산자를 사용해야 합니다.

```php
Co::set(['hook_flags'=> SWOOLE_HOOK_TCP | SWOOLE_HOOK_SLEEP]);
```

!> `Hook`된 함수는 [코루틴 컨테이너](/coroutine/scheduler)에서 사용되어야 합니다.
#### 자주 묻는 질문: ID = runtime-qa

!> **`Swoole\Runtime::enableCoroutine()` 및 `Co::set(['hook_flags'])` 중 어느 것을 사용해야 하는가**

* `Swoole\Runtime::enableCoroutine()`을 사용하면 서비스가 시작된 후(런타임)에도 flags를 동적으로 설정할 수 있으며, 해당 메서드를 호출한 후에는 현재 프로세스 전체에 영향을 미치며, 전체 프로젝트 시작 시에 100% 적용을 받기 위해 사용해야 합니다.
* `Co::set()`은 PHP의 `ini_set()`과 유사하게 이해할 수 있으며, [Server->start()](/server/methods?id=start) 또는 [Co\run()](/coroutine/scheduler) 전에 호출해야 합니다. 그렇지 않으면 설정된 `hook_flags`가 적용되지 않으며, `v4.4+` 버전에서는 이 방법으로 flags를 설정해야 합니다.
* `Co::set(['hook_flags'])` 또는 `Swoole\Runtime::enableCoroutine()` 모두 한 번만 호출해야 하며, 중복 호출 시 덮어씌워집니다.
## 옵션

`flags`가 지원하는 옵션은 다음과 같습니다:
```php
Co::set(['hook_flags' => SWOOLE_HOOK_ALL]); //CURL을 제외한 모든 유형의 플래그를 엽니다.
Co::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_CURL]); //실제로 모든 유형의 코루틴화, CURL 포함
```
### SWOOLE_HOOK_TCP

`v4.1`부터 지원되며, TCP 소켓 유형의 stream을 지원하며, 가장 흔한 `Redis`, `PDO`, `Mysqli`와 PHP의 [streams](https://www.php.net/streams) 시리즈 함수를 사용하여 TCP 연결을 처리하는 것도 `Hook` 할 수 있습니다. 샘플 코드:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TCP]);

Co\run(function() {
    for ($c = 100; $c--;) {
        go(function () {// 100 개의 코루틴 생성
            $redis = new Redis();
            $redis->connect('127.0.0.1', 6379);// 이 부분에서 코루틴 스케줄링이 발생하며, CPU는 다음 코루틴으로 전환되어 프로세스가 차단되지 않습니다
            $redis->get('key');// 이 부분에서 코루틴 스케줄링이 발생하며, CPU는 다음 코루틴으로 전환되어 프로세스가 차단되지 않습니다
        });
    }
});
```

위의 코드는 기본 `Redis` 클래스를 사용하지만 실제로 `비동기 IO`로 변환되었으며, `Co\run()`은 [코루틴 컨테이너](/coroutine/scheduler)를 만들고, `go()`는 코루틴을 만드는 것입니다. 이 두 작업은 `Swoole`이 제공하는 [Swoole\Server 클래스군](/server/init)에서 자동으로 수행되므로 직접 수행할 필요가 없습니다. [enable_coroutine](/server/setting?id=enable_coroutine)을 참고하세요.

즉, 전통적인 `PHP` 프로그래머가 익숙한 로직 코드로도 고성능 및 고성능 프로그램을 작성할 수 있습니다. 아래는:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TCP]);

$http = new Swoole\Http\Server("0.0.0.0", 9501);
$http->set(['enable_coroutine' => true]);

$http->on('request', function ($request, $response) {
      $redis = new Redis();
      $redis->connect('127.0.0.1', 6379); // 이 부분에서 코루틴 스케줄링이 발생하며, CPU는 다음 코루틴(다음 요청)으로 전환되어 프로세스가 차단되지 않습니다
      $redis->get('key'); // 이 부분에서 코루틴 스케줄링이 발생하며, CPU는 다음 코루틴(다음 요청)으로 전환되어 프로세스가 차단되지 않습니다
});

$http->start();
```
### SWOOLE_HOOK_UNIX

`v4.2`에서 지원됩니다. `Unix Stream Socket` 유형의 stream에 대한 예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UNIX]);

Co\run(function () {
    $socket = stream_socket_server(
        'unix://swoole.sock',
        $errno,
        $errstr,
        STREAM_SERVER_BIND | STREAM_SERVER_LISTEN
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_accept($socket)) {
    }
});
```
### SWOOLE_HOOK_UDP

`v4.2`부터 지원됩니다. UDP 소켓 유형의 스트림, 예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UDP]);

Co\run(function () {
    $socket = stream_socket_server(
        'udp://0.0.0.0:6666',
        $errno,
        $errstr,
        STREAM_SERVER_BIND
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_recvfrom($socket, 1, 0)) {
    }
});
```
### SWOOLE_HOOK_UDG

`v4.2`부터 지원됩니다. Unix Dgram Socket 유형의 stream, 예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UDG]);

Co\run(function () {
    $socket = stream_socket_server(
        'udg://swoole.sock',
        $errno,
        $errstr,
        STREAM_SERVER_BIND
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_recvfrom($socket, 1, 0)) {
    }
});
```
### SWOOLE_HOOK_SSL

`v4.2` 버전부터 지원됩니다. SSL 소켓 유형의 스트림 예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SSL]);

Co\run(function () {
    $host = 'host.domain.tld';
    $port = 1234;
    $timeout = 10;
    $cert = '/path/to/your/certchain/certchain.pem';
    $context = stream_context_create(
        array(
            'ssl' => array(
                'local_cert' => $cert,
            )
        )
    );
    if ($fp = stream_socket_client(
        'ssl://' . $host . ':' . $port,
        $errno,
        $errstr,
        30,
        STREAM_CLIENT_CONNECT,
        $context
    )) {
        echo "connected\n";
    } else {
        echo "ERROR: $errno - $errstr \n";
    }
});
```
### SWOOLE_HOOK_TLS

`v4.2`에서 지원됩니다. TLS 소켓 유형의 스트림으로, [참고](https://www.php.net/manual/en/context.ssl.php)하세요.

예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TLS]);
```
### SWOOLE_HOOK_SLEEP

`v4.2`부터 지원됩니다. `sleep` 함수의 `Hook`으로 `sleep`, `usleep`, `time_nanosleep`, `time_sleep_until`을 포함하며, 기본 타이머의 최소 단위가`1ms`이므로, `usleep`와 같은 고정밀 슬립 함수를 사용할 때,`1ms` 미만으로 설정된 경우, 직접적으로 `sleep` 시스템 호출을 사용합니다. 매우 짧은 슬립 블록을 유발할 수 있습니다. 예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SLEEP]);

Co\run(function () {
    go(function () {
        sleep(1);
        echo '1' . PHP_EOL;
    });
    go(function () {
        echo '2' . PHP_EOL;
    });
});
// 출력
2
1
```
### SWOOLE_HOOK_FILE

`v4.3`에서 지원됩니다.

* **파일 작업의 `코루틴 처리`, 지원하는 함수는 다음과 같습니다:**

    * `fopen`
    * `fread`/`fgets`
    * `fwrite`/`fputs`
    * `file_get_contents`、`file_put_contents`
    * `unlink`
    * `mkdir`
    * `rmdir`

예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_FILE]);

Co\run(function () {
    $fp = fopen("test.log", "a+");
    fwrite($fp, str_repeat('A', 2048));
    fwrite($fp, str_repeat('B', 2048));
});
```
### SWOOLE_HOOK_STREAM_FUNCTION

`v4.4`부터 지원됩니다. `stream_select()`의 `Hook` 예제:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_STREAM_FUNCTION]);

Co\run(function () {
    $fp1 = stream_socket_client("tcp://www.baidu.com:80", $errno, $errstr, 30);
    $fp2 = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
    if (!$fp1) {
        echo "$errstr ($errno) \n";
    } else {
        fwrite($fp1, "GET / HTTP/1.0\r\nHost: www.baidu.com\r\nUser-Agent: curl/7.58.0\r\nAccept: */*\r\n\r\n");
        $r_array = [$fp1, $fp2];
        $w_array = $e_array = null;
        $n = stream_select($r_array, $w_array, $e_array, 10);
        $html = '';
        while (!feof($fp1)) {
            $html .= fgets($fp1, 1024);
        }
        fclose($fp1);
    }
});
```
### SWOOLE_HOOK_BLOCKING_FUNCTION

`v4.4`부터 지원됩니다. 여기서 'blocking function'은 `gethostbyname`, `exec`, `shell_exec` 등을 포함합니다. 예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_BLOCKING_FUNCTION]);

Co\run(function () {
    echo shell_exec('ls');
});
```
### SWOOLE_HOOK_PROC

`v4.4` 에서 지원합니다. `proc*` 함수를 코루틴화하는데 사용됩니다. `proc_open`, `proc_close`, `proc_get_status`, `proc_terminate` 등이 있습니다.

예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PROC]);

Co\run(function () {
    $descriptorspec = array(
        0 => array("pipe", "r"),  // stdin, 자식 프로세스가 여기서 읽음
        1 => array("pipe", "w"),  // stdout, 자식 프로세스가 여기에 쓰기
    );
    $process = proc_open('php', $descriptorspec, $pipes);
    if (is_resource($process)) {
        fwrite($pipes[0], '<?php echo "I am process\n" ?>');
        fclose($pipes[0]);

        while (true) {
            echo fread($pipes[1], 1024);
        }

        fclose($pipes[1]);
        $return_value = proc_close($process);
        echo "command returned $return_value" . PHP_EOL;
    }
});
```
### SWOOLE_HOOK_CURL

[v4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x) 이후 또는 `v4.5`부터 공식 지원됩니다.

- **CURL 훅을 지원하는 함수:**
    - curl_init
    - curl_setopt
    - curl_exec
    - curl_multi_getcontent
    - curl_setopt_array
    - curl_error
    - curl_getinfo
    - curl_errno
    - curl_close
    - curl_reset

예시:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_CURL]);

Co\run(function () {
    $ch = curl_init();  
    curl_setopt($ch, CURLOPT_URL, "http://www.xinhuanet.com/");  
    curl_setopt($ch, CURLOPT_HEADER, false);  
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $result = curl_exec($ch);  
    curl_close($ch);
    var_dump($result);
});
```
### SWOOLE_HOOK_NATIVE_CURL

`Coroutine handling` for native CURL.

!> Available in Swoole version >= `v4.6.0`

!> Before using, the [--enable-swoole-curl](/environment?id=common-parameters) option must be enabled during compilation;  
After enabling this option, `SWOOLE_HOOK_NATIVE_CURL` will be automatically set, and `SWOOLE_HOOK_CURL` will be disabled;  
Also, `SWOOLE_HOOK_ALL` includes `SWOOLE_HOOK_NATIVE_CURL`

```php
Co::set(['hook_flags' => SWOOLE_HOOK_NATIVE_CURL]);

Co::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_NATIVE_CURL]);
```

Example:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_ALL]);

Co\run(function () {
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, "http://httpbin.org/get");
    curl_setopt($ch, CURLOPT_HEADER, false);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $result = curl_exec($ch);
    curl_close($ch);
    var_dump($result);
});
```
### SWOOLE_HOOK_SOCKETS

소켓 확장에 대한 `코루틴 처리`입니다.

!> Swoole 버전 >= `v4.6.0` 에서 사용 가능합니다

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SOCKETS]);
```
### SWOOLE_HOOK_STDIO

`STDIO`의 코루틴 처리입니다.

!> Swoole 버전 >= `v4.6.2`에서 사용 가능

```php
Co::set(['hook_flags' => SWOOLE_HOOK_STDIO]);
```

예시:

```php
use Swoole\Process;
Co::set(['socket_read_timeout' => -1, 'hook_flags' => SWOOLE_HOOK_STDIO]);
$proc = new Process(function ($p) {
    Co\run(function () use($p) {
        $p->write('start'.PHP_EOL);
        go(function() {
            co::sleep(0.05);
            echo "sleep\n";
        });
        echo fread(STDIN, 1024);
    });
}, true, SOCK_STREAM);
$proc->start();
echo $proc->read();
usleep(100000);
$proc->write('hello world'.PHP_EOL);
echo $proc->read();
echo $proc->read();
Process::wait();
```
### SWOOLE_HOOK_PDO_PGSQL

`pdo_pgsql`에 대한 `코루틴 처리`입니다.

!> Swoole 버전 >= `v5.1.0`에서 사용 가능

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_PGSQL]);
```

예시:
```php
<?php
function test()
{
    $dbname   = "test";
    $username = "test";
    $password = "test";
    try {
        $dbh = new PDO("pgsql:dbname=$dbname;host=127.0.0.1:5432", $username, $password);
        $dbh->exec('create table test (id int)');
        $dbh->exec('insert into test values(1)');
        $dbh->exec('insert into test values(2)');
        $res = $dbh->query("select * from test");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['trace_flags' => SWOOLE_HOOK_PDO_PGSQL]);

Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_ODBC

`pdo_odbc`의 `코루틴 처리`입니다.

!> Swoole 버전 >= `v5.1.0` 에서 사용 가능

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ODBC]);
```

예시:
```php
<?php
function test()
{
    $username = "test";
    $password = "test";
    try {
        $dbh = new PDO("odbc:mysql-test");
        $res = $dbh->query("select sleep(1) s");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['trace_flags' => SWOOLE_TRACE_CO_ODBC, 'log_level' => SWOOLE_LOG_DEBUG]);

Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_ORACLE

`pdo_oci`의 코루틴 처리를 위한 것입니다.

!> Swoole 버전 >= `v5.1.0`에서 사용 가능합니다.

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ORACLE]);
```

예시:
```php
<?php
function test()
{
	$tsn = 'oci:dbname=127.0.0.1:1521/xe;charset=AL32UTF8';
	$username = "test";
	$password = "test";
    try {
        $dbh = new PDO($tsn, $username, $password);
        $dbh->exec('create table test (id int)');
        $dbh->exec('insert into test values(1)');
        $dbh->exec('insert into test values(2)');
        $res = $dbh->query("select * from test");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ORACLE]);
Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_SQLITE
`pdo_sqlite`의 `코루틴 처리`.

!> Swoole version >= `v5.1.0`에서 사용 가능

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_SQLITE]);
```

* **주의사항**

!> `swoole`은 `코루틴`에서 `sqlite` 데이터베이스를 처리할 때 [스레드 안전성](https://www.sqlite.org/threadsafe.html)을 보장하기 위해 `직렬화 모드`를 사용합니다.  
만약 `sqlite` 데이터베이스를 컴파일할 때 단일 스레드 모드로 지정했다면, `swoole`은 `sqlite`를 코루틴으로 처리할 수 없으며 경고가 발생하지만 사용에는 영향을 미치지 않습니다. 단순히 삽입, 삭제, 수정, 검색 과정에서 코루틴 전환이 발생하지 않습니다. 이러한 경우에는 `sqlite`를 다시 컴파일하고 스레드 모드를 `직렬화` 또는 `다중 스레드`로 지정해야 합니다. [원인](https://www.sqlite.org/compile.html#threadsafe)   
코루틴 환경에서 생성된 `sqlite` 연결은 모두 `직렬화`됩니다. 비 코루틴 환경에서 생성된 `sqlite` 연결은 기본적으로 `sqlite`의 스레드 모드와 일치합니다.  
만약 `sqlite`의 스레드 모드가 `다중 스레드`인 경우, 비 코루틴 환경에서 생성된 연결은 여러 코루틴과 공유할 수 없습니다. 이는 데이터베이스 연결이 `다중 스레드 모드`이기 때문에 코루틴 환경에서 사용해도 `직렬화`로 업그레이드되지 않습니다.  
`sqlite`의 기본 스레드 모드는 `직렬화`입니다. [직렬화 설명](https://www.sqlite.org/c3ref/c_config_covering_index_scan.html#sqliteconfigserialized), [기본 스레드 모드](https://www.sqlite.org/compile.html#threadsafe)   

예제:
```php
<?php
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\go;

Co::set(['hook_flags'=> SWOOLE_HOOK_PDO_SQLITE]);

run(function() {
    for($i = 0; $i <= 5; $i++) {
        go(function() use ($i) {
            $db = new PDO('sqlite::memory:');
            $db->query('select randomblob(99999999)');
            var_dump($i);
        });
    }
});
```
```python
def greet():
    return "Hello!"
```

위의 코드는 간단한 인사말을 출력하는 함수를 정의하는 것입니다.
```php
Swoole\Runtime::setHookFlags(int $flags): bool
```
### getHookFlags()

현재 `Hook` 된 내용의 `flags`를 가져옵니다. 이는 `Hook`을 할 때 전달된 `flags`와 일치하지 않을 수 있습니다. (실패한 `flags`는 지워질 수 있음)

!> Swoole 버전 >= `v4.4.12`에서 사용 가능

```php
Swoole\Runtime::getHookFlags(): int
```
## 흔한 Hook 목록
### 사용 가능한 목록

  * `redis` 확장
  * `mysqlnd` 모드의 `pdo_mysql`, `mysqli` 확장 사용, `mysqlnd`가 비활성화되면 코루틴 처리가 지원되지 않음
  * `soap` 확장
  * `file_get_contents`, `fopen`
  * `stream_socket_client` (`predis`, `php-amqplib`)
  * `stream_socket_server`
  * `stream_select` (버전 `4.3.2` 이상 필요)
  * `fsockopen`
  * `proc_open` (버전 `4.4.0` 이상 필요)
  * `curl`
### 사용 불가 목록

!> **코루틴을 지원하지 않음**은 코루틴을 블로킹 모드로 강등시키므로, 해당 시나리오에서는 코루틴을 사용하는 것이 의미가 없음

  * `mysql`：하위 단계에서 `libmysqlclient`를 사용
  * `mongo`：하위 단계에서 `mongo-c-client`를 사용
  * `pdo_pgsql`，Swoole 버전이 `v5.1.0` 이상인 경우에는 `pdo_pgsql`을 사용하여 코루틴 처리가 가능
  * `pdo_oci`，Swoole 버전이 `v5.1.0` 이상인 경우에는 `pdo_oci`를 사용하여 코루틴 처리가 가능
  * `pdo_odbc`，Swoole 버전이 `v5.1.0` 이상인 경우에는 `pdo_odbc`를 사용하여 코루틴 처리가 가능
  * `pdo_firebird`
  * `php-amqp`
## API 변경

`v4.3` 및 이전 버전에서는 `enableCoroutine`의 API가 2개의 매개변수를 필요로 합니다.

```php
Swoole\Runtime::enableCoroutine(bool $enable = true, int $flags = SWOOLE_HOOK_ALL);
```

- `$enable` : 코루틴 기능을 켜거나 끕니다.
- `$flags` : 선택사항으로 `코루틴 활성화`의 유형을 선택하며, 여러 개를 선택할 수 있으며 기본값은 모두 선택입니다. `$enable = true`일 때만 유효합니다.

!> `Runtime::enableCoroutine(false)`는 이전에 설정된 모든 옵션 코루틴 `Hook` 설정을 닫습니다.
