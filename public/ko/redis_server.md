# Redis\Server

`Server` 클래스는 `Redis` 서버 프로토콜과 호환되며, 이 클래스를 기반으로 `Redis` 프로토콜을 따르는 서버 프로그램을 구현할 수 있습니다.

?> `Swoole\Redis\Server`는 [Server](/server/tcp_init)을 상속받았으므로, `Server`가 제공하는 모든 `API` 및 설정을 사용할 수 있으며, 프로세스 모델도 동일합니다. 자세한 내용은 [Server](/server/init) 섹션을 참조하세요.

* **사용 가능한 클라이언트**

  * 모든 프로그래밍 언어의 `redis` 클라이언트, PHP의 `redis` 확장 및 `phpredis` 라이브러리 포함
  * [Swoole\Coroutine\Redis](/coroutine_client/redis) 코루틴 클라이언트
  * `Redis`에서 제공하는 명령행 도구인 `redis-cli`, `redis-benchmark` 포함
## 메소드

`Swoole\Redis\Server`는 `Swoole\Server`를 상속받았으며, 부모 클래스가 제공하는 모든 메소드를 사용할 수 있습니다.
### setHandler

?> **'Redis' 명령어 처리기를 설정합니다.**

!> `Redis\Server`에서는 [onReceive](/server/events?id=onreceive) 콜백을 설정할 필요가 없습니다. 단순히 해당 명령에 대한 처리기 함수를 `setHandler` 메서드를 사용하여 설정하면, 지원되지 않는 명령을 수신하면 자동으로 클라이언트에 `ERROR` 응답을 보냅니다. 메시지는 `ERR unknown command '$command'`로 표시됩니다.

```php
Swoole\Redis\Server->setHandler(string $command, callable $callback);
```

* **파라미터** 

  * **`string $command`**
    * **기능** : 명령어의 이름
    * **기본값** : 없음
    * **다른 값** : 없음

  * **`callable $callback`**
    * **기능** : 명령어의 처리 함수【콜백 함수가 문자열을 반환하면 클라이언트로 자동 전송됨】
    * **기본값** : 없음
    * **다른 값** : 없음

    !> 반환되는 데이터는 `Redis` 형식이어야 하며, 패키징하기 위해 `format` 정적 메서드를 사용할 수 있음
### 형식

?> **응답 데이터를 형식화합니다.**

```php
Swoole\Redis\Server::format(int $type, mixed $value = null);
```

* **매개변수** 

  * **`int $type`**
    * **기능**：데이터 유형으로, 아래 포맷 매개변수 상수를 참조하세요 [포맷 매개변수 상수](/redis_server?id=포맷-매개변수-상수)。
    * **기본값**：없음
    * **다른 값**：없음
    
    !> `NIL` 유형일 때는 `$value`를 전달할 필요가 없습니다. `ERROR` 및 `STATUS` 유형에서는`$value`가 선택 사항이며 `INT` `STRING` `SET` 및 `MAP`은 필수입니다.

  * **`mixed $value`**
    * **기능**：값
    * **기본값**：없음
    * **다른 값**：없음
### send

?> **`send()` 메서드를 사용하여 데이터를 클라이언트에 전송합니다.**

```php
Swoole\Server->send(int $fd, string $data): bool
```
## 상수
### 형식 매개변수 상수

`format` 함수를 사용하여 `Redis` 응답 데이터를 패킹하는 데 사용됩니다

상수 | 설명
---|---
Server::NIL | nil 데이터 반환
Server::ERROR | 오류 코드 반환
Server::STATUS | 상태 반환
Server::INT | 정수 반환, format은 매개변수 값이어야 하며, 유형은 정수여야 함
Server::STRING | 문자열 반환, format은 매개변수 값이어야 하며, 유형은 문자열이어야 함
Server::SET | 리스트 반환, format은 매개변수 값이어야 하며, 유형은 배열이어야 함
Server::MAP | Map 반환, format은 매개변수 값이어야 하며, 유형은 연관 배열이어야 함
```python
def add_numbers(x, y):
    return x + y

result = add_numbers(3, 5)
print(result)  # Output: 8
```

위의 코드는 두 숫자를 더하는 간단한 함수를 보여줍니다. 결과값을 출력할 때 `8`이 나옵니다.
### 서버 측

```php
use Swoole\Redis\Server;

define('DB_FILE', __DIR__ . '/db');

$server = new Server("127.0.0.1", 9501, SWOOLE_BASE);

if (is_file(DB_FILE)) {
    $server->data = unserialize(file_get_contents(DB_FILE));
} else {
    $server->data = array();
}

$server->setHandler('GET', function ($fd, $data) use ($server) {
    if (count($data) == 0) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR 'GET' 명령어에 대한 잘못된 인수 수"));
    }

    $key = $data[0];
    if (empty($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    } else {
        return $server->send($fd, Server::format(Server::STRING, $server->data[$key]));
    }
});

$server->setHandler('SET', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR 'SET' 명령어에 대한 잘못된 인수 수"));
    }

    $key = $data[0];
    $server->data[$key] = $data[1];
    return $server->send($fd, Server::format(Server::STATUS, "OK"));
});

$server->setHandler('sAdd', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR 'sAdd' 명령어에 대한 잘못된 인수 수"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }

    $count = 0;
    for ($i = 1; $i < count($data); $i++) {
        $value = $data[$i];
        if (!isset($server->data[$key][$value])) {
            $server->data[$key][$value] = 1;
            $count++;
        }
    }

    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('sMembers', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR 'sMembers' 명령어에 대한 잘못된 인수 수"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::SET, array_keys($server->data[$key])));
});

$server->setHandler('hSet', function ($fd, $data) use ($server) {
    if (count($data) < 3) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR 'hSet' 명령어에 대한 잘못된 인수 수"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }
    $field = $data[1];
    $value = $data[2];
    $count = !isset($server->data[$key][$field]) ? 1 : 0;
    $server->data[$key][$field] = $value;
    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('hGetAll', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR 'hGetAll' 명령어에 대한 잘못된 인수 수"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::MAP, $server->data[$key]));
});

$server->on('WorkerStart', function ($server) {
    $server->tick(10000, function () use ($server) {
        file_put_contents(DB_FILE, serialize($server->data));
    });
});

$server->start();
```
### 클라이언트

```shell
$ redis-cli -h 127.0.0.1 -p 9501
127.0.0.1:9501> set name swoole
OK
127.0.0.1:9501> get name
"swoole"
127.0.0.1:9501> sadd swooler rango
(integer) 1
127.0.0.1:9501> sadd swooler twosee guoxinhua
(integer) 2
127.0.0.1:9501> smembers swooler
1) "rango"
2) "twosee"
3) "guoxinhua"
127.0.0.1:9501> hset website swoole "www.swoole.com"
(integer) 1
127.0.0.1:9501> hset website swoole "swoole.com"
(integer) 0
127.0.0.1:9501> hgetall website
1) "swoole"
2) "swoole.com"
127.0.0.1:9501> test
(error) ERR unknown command 'test'
127.0.0.1:9501>
```
