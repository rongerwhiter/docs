# 코루틴 Redis 클라이언트

!> 이 클라이언트는 더 이상 권장되지 않으며, `Swoole\Runtime::enableCoroutine + phpredis` 또는 `predis`를 사용하는 것을 권장합니다. '원클릭 코루틴 활성화'를 통해 네이티브 PHP의 redis 클라이언트를 사용하세요.
## 사용 예시

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $val = $redis->get('key');
});
```

!> `subscribe` 및 `pSubscribe`는 `defer(true)` 상황에서 사용할 수 없습니다.
## 방법

!> 방법은 기본적으로 [phpredis](https://github.com/phpredis/phpredis)와 동일하게 사용됩니다.

다음은 [phpredis](https://github.com/phpredis/phpredis)와 다른 구현 사항입니다:

1. 아직 구현되지 않은 Redis 명령어: `scan object sort migrate hscan sscan zscan`;

2. `subscribe pSubscribe`의 사용 방법에 대한 차이로, 콜백 함수를 설정할 필요가 없습니다;

3. PHP 변수 직렬화 지원은 `connect()` 메소드의 세 번째 매개변수를 `true`로 설정하면 활성화됩니다. 기본값은 `false`입니다.
### __construct()

`Redis` 연결 구성 옵션을 설정할 수 있는 Redis 코루틴 클라이언트 생성자, `setOptions()` 메서드와 동일한 매개변수를 사용합니다.

```php
Swoole\Coroutine\Redis::__construct(array $options = null);
```
### setOptions()

4.2.10 버전 이후에 추가된 이 메서드는 `Redis` 클라이언트의 설정을 구성하는 데 사용됩니다.

이 함수는 Swoole 스타일이며 `Key-Value` 페어 배열을 통해 구성해야 합니다.

```php
Swoole\Coroutine\Redis->setOptions(array $options): void
```

  * **구성 가능한 옵션**

key | 설명
---|---
`connect_timeout` | 연결 시간 초과, 기본값은 글로벌 코루틴 `socket_connect_timeout`(1초)
`timeout` | 시간 초과, 기본값은 글로벌 코루틴 `socket_timeout`, [클라이언트 시간 초과 규칙](/coroutine_client/init?id=시간초과 규칙) 참조
`serialize` | 자동 직렬화, 기본적으로 비활성화
`reconnect` | 자동 연결 시도 횟수, 연결이 시간 초과 등으로 인해 `close`로 정상적으로 끊기면, 다음 요청 시 자동으로 연결을 시도한 후 요청을 보냅니다. 기본값은 1회(`true`), 지정된 회수만큼 시도한 후 더 이상 시도하지 않고 수동으로 다시 연결해야 합니다. 이 메커니즘은 연결 유지를 위한 것으로, 요청을 재전송하여 무효한 인터페이스 오류 등을 발생시키지 않습니다.
`compatibility_mode` | `hmGet/hGetAll/zRange/zRevRange/zRangeByScore/zRevRangeByScore` 함수의 반환 결과가 `php-redis`와 일치하지 않는 호환성 문제 해결 방법입니다. 이 옵션을 활성화하면 `Co\Redis`와 `php-redis`가 동일한 결과를 반환합니다. 기본적으로 비활성화됩니다. 【이 구성 항목은`v4.4.0` 이상에서 사용할 수 있습니다】
### set()

데이터를 저장합니다.

```php
Swoole\Coroutine\Redis->set(string $key, mixed $value, array|int $option): bool
```

  * **매개변수** 

    * **`string $key`**
      * **기능**：데이터의 키
      * **기본값**：없음
      * **다른 값들**：없음

    * **`string $value`**
      * **기능**：데이터 내용【문자열이 아닌 유형은 자동 직렬화됨】
      * **기본값**：없음
      * **다른 값들**：없음

    * **`string $options`**
      * **기능**：옵션
      * **기본값**：없음
      * **다른 값들**：없음

      !> `$option` 설명：  
      `정수값`：만료 시간 설정, 예: `3600`  
      `배열`：고급 만료 설정, 예: `['nx', 'ex' => 10]` 、 `['xx', 'px' => 1000]`

      !> `px`: 밀리초 단위로 만료 시간 표시  
      `ex`: 초 단위로 만료 시간 표시  
      `nx`: 존재하지 않을 때에만 타임아웃 설정  
      `xx`: 존재할 때에만 타임아웃 설정
### request()

Redis 서버에 사용자 정의 명령을 보냅니다. 이는 phpredis의 rawCommand와 유사합니다.

```php
Swoole\Coroutine\Redis->request(array $args): void
```

  * **Parameters** 

    * **`array $args`**
      * **Description**: Arguments list, must be in array format.【The first element must be the `Redis` command, and the rest are command parameters which will be automatically packed into a `Redis` protocol request for sending.】
      * **Default**: None
      * **Other values**: None

  * **Return** 

Depends on how the `Redis` server handles the command, it may return types such as numbers, booleans, strings, arrays, etc.

  * **Example** 

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379); // If using local UNIX socket, the host parameter should be in the format like `unix://tmp/your_file.sock`
    $res = $redis->request(['object', 'encoding', 'key1']);
    var_dump($res);
});
```
## 속성
### errCode

오류 코드.

오류 코드 | 설명
---|---
1 | 읽기 또는 쓰기 오류
2 | 나머지 모든 것...
3 | 파일 끝
4 | 프로토콜 오류
5 | 메모리 부족
### errMsg

오류 메시지.
### connected

현재 `Redis` 클라이언트가 서버에 연결되어 있는지 여부를 판단합니다.
## 상수

`multi($mode)` 메소드의 기본 값은 `SWOOLE_REDIS_MODE_MULTI` 모드입니다:

* SWOOLE_REDIS_MODE_MULTI
* SWOOLE_REDIS_MODE_PIPELINE

`type()` 명령의 반환 값을 판단하는 데 사용됩니다:

* SWOOLE_REDIS_TYPE_NOT_FOUND
* SWOOLE_REDIS_TYPE_STRING
* SWOOLE_REDIS_TYPE_SET
* SWOOLE_REDIS_TYPE_LIST
* SWOOLE_REDIS_TYPE_ZSET
* SWOOLE_REDIS_TYPE_HASH
## 트랜잭션 모드

`Redis`의 트랜잭션 모드는 `multi` 및 `exec`를 사용하여 구현할 수 있습니다.

  * **팁**

    * `multi` 명령어를 사용하여 트랜잭션을 시작하고, 그 이후에 모든 명령어는 대기열에 추가되어 실행을 기다립니다.
    * `exec` 명령어를 사용하여 트랜잭션 내의 모든 작업을 실행하고, 모든 결과를 한 번에 반환합니다.

  * **사용 예시**

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $redis->multi();
    $redis->set('key3', 'rango');
    $redis->get('key1');
    $redis->get('key2');
    $redis->get('key3');

    $result = $redis->exec();
    var_dump($result);
});
```
## 구독 모드

!> Swoole 버전 >= v4.2.13에서 사용 가능합니다. **4.2.12 및 그 이하 버전에서는 구독 모드에 버그가 있습니다.**
### 구독

`phpredis`와는 다르게 `subscribe/psubscribe`는 코루틴 스타일을 사용합니다.

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    if ($redis->subscribe(['channel1', 'channel2', 'channel3'])) // 또는 psubscribe를 사용할 수도 있음
    {
        while ($msg = $redis->recv()) {
            // msg는 배열이며 아래 정보를 포함합니다
            // $type # 반환 값의 유형: 성공적으로 구독했다는 메시지
            // $name # 구독한 채널 이름 또는 원본 채널 이름
            // $info  # 현재 구독 중인 채널 수 또는 정보 내용
            list($type, $name, $info) = $msg;
            if ($type == 'subscribe') { // 또는 psubscribe
                // 채널 구독 성공 메시지, 구독한 채널 수만큼 메시지 수신
            } else if ($type == 'unsubscribe' && $info == 0){ // 또는 punsubscribe
                break; // 구독 취소 메시지를 받았고 남은 구독 채널 수가 0인 경우 더 이상 메시지를 받지 않고 루프 종료
            } else if ($type == 'message') {  // psubscribe인 경우 여기는 pmessage가 됨
                var_dump($name); // 원본 채널 이름 출력
                var_dump($info); // 메시지 출력
                // 메시지 처리
                if ($need_unsubscribe) { // 특정 상황에서 구독을 취소해야 할 경우
                    $redis->unsubscribe(); // 취소가 완료될 때까지 recv를 계속 기다림
                }
            }
        }
    }
});
```
### 구독 취소

`unsubscribe/punsubscribe`을 사용하여 구독을 취소할 수 있습니다. `$redis->unsubscribe(['channel1'])`

이때, `$redis->recv()`를 통해 구독 취소 메시지를 수신하게 됩니다. 여러 채널을 구독 취소하는 경우, 여러 메시지를 받게 됩니다.

!> 참고: 구독 취소 후에는 반드시 마지막 구독 취소 메시지(`$msg[2] == 0`)를 수신할 때까지 `recv()`를 계속해야 합니다. 이 메시지를 받은 이후에야 구독 모드를 종료합니다.

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    if ($redis->subscribe(['channel1', 'channel2', 'channel3'])) // 혹은 psubscribe를 사용할 수도 있음
    {
        while ($msg = $redis->recv()) {
            // 메시지는 다음 정보를 포함하는 배열입니다
            // $type # 반환 형식: 구독 성공 표시
            // $name # 구독된 채널 이름 또는 소스 채널 이름
            // $info  # 현재 구독된 채널 수 또는 정보 콘텐츠
            list($type, $name, $info) = $msg;
            if ($type == 'subscribe') // 혹은 psubscribe
            {
                // 채널 구독 성공 메시지
            }
            else if ($type == 'unsubscribe' && $info == 0) // 혹은 punsubscribe
            {
                break; // 구독 취소 메시지를 받고, 남아 있는 채널 수가 0인 경우 반복문 종료
            }
            else if ($type == 'message') // psubscribe인 경우 여기는 pmessage
            {
                // 소스 채널 이름 출력
                var_dump($name);
                // 메시지 출력
                var_dump($info);
                // 메시지 처리
                if ($need_unsubscribe) // 경우에 따라 구독을 취소해야 할 수도 있음
                {
                    $redis->unsubscribe(); // 구독이 완료될 때까지 recv를 계속
                }
            }
        }
    }
});
```
## 호환 모드

`Co\Redis`의 `hmGet/hGetAll/zrange/zrevrange/zrangebyscore/zrevrangebyscore` 명령이 `phpredis` 확장의 반환 형식과 일치하지 않는 문제가 해결되었습니다 [#2529](https://github.com/swoole/swoole-src/pull/2529).

이전 버전과 호환성을 유지하기 위해 `$redis->setOptions(['compatibility_mode' => true]);` 설정을 추가하면 `Co\Redis`와 `phpredis`가 동일한 결과를 반환합니다.

!> Swoole 버전 >= `v4.4.0`에서 사용 가능

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->setOptions(['compatibility_mode' => true]);
    $redis->connect('127.0.0.1', 6379);

    $co_get_val = $redis->get('novalue');
    $co_zrank_val = $redis->zRank('novalue', 1);
    $co_hgetall_val = $redis->hGetAll('hkey');
    $co_hmget_val = $redis->hmGet('hkey', array(3, 5));
    $co_zrange_val = $redis->zRange('zkey', 0, 99, true);
    $co_zrevrange_val = $redis->zRevRange('zkey', 0, 99, true);
    $co_zrangebyscore_val = $redis->zRangeByScore('zkey', 0, 99, ['withscores' => true]);
    $co_zrevrangebyscore_val = $redis->zRevRangeByScore('zkey', 99, 0, ['withscores' => true]);
});
```
