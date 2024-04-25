# 연결 풀

Swoole은`v4.4.13` 버전부터 내장 협동 연결 풀을 제공합니다. 이 섹션은 해당 연결 풀을 사용하는 방법에 대해 설명합니다.
## ConnectionPool

[ConnectionPool](https://github.com/swoole/library/blob/master/src/core/ConnectionPool.php)는 채널을 기반으로 하는 원시 연결 풀로, 임의의 생성자(`callable`)를 전달할 수 있습니다. 생성자는 연결 개체를 반환해야 합니다.

* `get` 메서드는 연결을 가져옵니다 (연결 풀이 가득 차지 않은 경우 새로운 연결을 만듭니다).
* `put` 메서드는 연결을 회수합니다.
* `fill` 메서드는 연결 풀을 채웁니다 (연결을 미리 생성합니다).
* `close`는 연결 풀을 닫습니다.

!> [Simps 프레임워크](https://simps.io)의 [DB 구성요소](https://github.com/simple-swoole/db)는 Database를 기반으로하여 자동으로 연결을 되돌려주고 트랜잭션 등의 기능을 구현했습니다. 참고 또는 직접 사용할 수 있으며 자세한 내용은 [Simps 문서](https://simps.io/#/zh-cn/database/mysql)를 참조하세요.
## 데이터베이스

다양한 데이터베이스 연결 풀 및 객체 프록시의 고급 래핑으로, 자동 재연결을 지원합니다. 현재 PDO, Mysqli, Redis 세 가지 유형의 데이터베이스 지원이 포함되어 있습니다:

* `PDOConfig`, `PDOProxy`, `PDOPool`
* `MysqliConfig`, `MysqliProxy`, `MysqliPool`
* `RedisConfig`, `RedisProxy`, `RedisPool`

!> 1. MySQL 재연결은 대부분의 연결 컨텍스트(페치 모드, 설정된 attribute, 컴파일 된 명령문 등)를 자동으로 복원하지만 트랜잭션과 같은 컨텍스트는 복원할 수 없습니다. 트랜잭션 중에 연결이 끊어진 경우 예외가 발생하므로 재연결의 신뢰성을 직접 평가하셔야 합니다.  
2. 트랜잭션 중인 연결을 연결 풀로 반환하는 것은 정의되지 않은 동작입니다. 개발자는 반환되는 연결이 재사용 가능하도록 보장해야 합니다.  
3. 예외가 발생하여 재사용할 수 없는 연결 객체가 있는 경우, 개발자는 `$pool->put(null);`을 호출하여 빈 연결을 반환하여 연결 풀의 수를 유지해야 합니다.
### PDOPool/MysqliPool/RedisPool :id=pool

연결 풀 객체를 만드는 데 사용되며 두 개의 매개변수가 존재하며 각각 해당 Config 객체와 연결 풀 크기입니다

```php
$pool = new \Swoole\Database\PDOPool(Swoole\Database\PDOConfig $config, int $size);

$pool = new \Swoole\Database\MysqliPool(Swoole\Database\MysqliConfig $config, int $size);

$pool = new \Swoole\Database\RedisPool(Swoole\Database\RedisConfig $config, int $size);
```

  * **Parameters** 

    * **`$config`**
      * **Purpose**：해당 Config 객체, 자세한 사용법은 아래의 [사용 예시](/coroutine/conn_pool?id=사용 예시) 참조
      * **Default Value**：없음
      * **Other Values**：【[PDOConfig](https://github.com/swoole/library/blob/master/src/core/Database/PDOConfig.php)、[RedisConfig](https://github.com/swoole/library/blob/master/src/core/Database/RedisConfig.php)、[MysqliConfig](https://github.com/swoole/library/blob/master/src/core/Database/MysqliConfig.php)】
      
    * **`int $size`**
      * **Purpose**：연결 풀 크기
      * **Default Value**：64
      * **Other Values**：없음
번역할 텍스트가 없습니다.
```php
<?php
declare(strict_types=1);

use Swoole\Coroutine;
use Swoole\Database\PDOConfig;
use Swoole\Database\PDOPool;
use Swoole\Runtime;

const N = 1024;

Runtime::enableCoroutine();
$s = microtime(true);
Coroutine\run(function () {
    $pool = new PDOPool((new PDOConfig)
        ->withHost('127.0.0.1')
        ->withPort(3306)
        // ->withUnixSocket('/tmp/mysql.sock')
        ->withDbName('test')
        ->withCharset('utf8mb4')
        ->withUsername('root')
        ->withPassword('root')
    );
    for ($n = N; $n--;) {
        Coroutine::create(function () use ($pool) {
            $pdo = $pool->get();
            $statement = $pdo->prepare('SELECT ? + ?');
            if (!$statement) {
                throw new RuntimeException('준비 실패');
            }
            $a = mt_rand(1, 100);
            $b = mt_rand(1, 100);
            $result = $statement->execute([$a, $b]);
            if (!$result) {
                throw new RuntimeException('실행 실패');
            }
            $result = $statement->fetchAll();
            if ($a + $b !== (int)$result[0][0]) {
                throw new RuntimeException('잘못된 결과');
            }
            $pool->put($pdo);
        });
    }
});
$s = microtime(true) - $s;
echo '쿼리 ' . N . ' 개에 대해 ' . $s . '초 사용' . PHP_EOL;
```
```php
<?php
declare(strict_types=1);

use Swoole\Coroutine;
use Swoole\Database\RedisConfig;
use Swoole\Database\RedisPool;
use Swoole\Runtime;

const N = 1024;

Runtime::enableCoroutine();
$s = microtime(true);
Coroutine\run(function () {
    $pool = new RedisPool((new RedisConfig)
        ->withHost('127.0.0.1')
        ->withPort(6379)
        ->withAuth('')
        ->withDbIndex(0)
        ->withTimeout(1)
    );
    for ($n = N; $n--;) {
        Coroutine::create(function () use ($pool) {
            $redis = $pool->get();
            $result = $redis->set('foo', 'bar');
            if (!$result) {
                throw new RuntimeException('Set failed');
            }
            $result = $redis->get('foo');
            if ($result !== 'bar') {
                throw new RuntimeException('Get failed');
            }
            $pool->put($redis);
        });
    }
});
$s = microtime(true) - $s;
echo 'Use ' . $s . 's for ' . (N * 2) . ' queries' . PHP_EOL;
```
```php
<?php
declare(strict_types=1);

use Swoole\Coroutine;
use Swoole\Database\MysqliConfig;
use Swoole\Database\MysqliPool;
use Swoole\Runtime;

const N = 1024;

Runtime::enableCoroutine();
$s = microtime(true);
Coroutine\run(function () {
    $pool = new MysqliPool((new MysqliConfig)
        ->withHost('127.0.0.1')
        ->withPort(3306)
        // ->withUnixSocket('/tmp/mysql.sock')
        ->withDbName('test')
        ->withCharset('utf8mb4')
        ->withUsername('root')
        ->withPassword('root')
    );
    for ($n = N; $n--;) {
        Coroutine::create(function () use ($pool) {
            $mysqli = $pool->get();
            $statement = $mysqli->prepare('SELECT ? + ?');
            if (!$statement) {
                throw new RuntimeException('준비에 실패했습니다.');
            }
            $a = mt_rand(1, 100);
            $b = mt_rand(1, 100);
            if (!$statement->bind_param('dd', $a, $b)) {
                throw new RuntimeException('바인딩 파라미터에 실패했습니다.');
            }
            if (!$statement->execute()) {
                throw new RuntimeException('실행에 실패했습니다.');
            }
            if (!$statement->bind_result($result)) {
                throw new RuntimeException('결과 바인딩에 실패했습니다.');
            }
            if (!$statement->fetch()) {
                throw new RuntimeException('검색에 실패했습니다.');
            }
            if ($a + $b !== (int)$result) {
                throw new RuntimeException('잘못된 결과입니다.');
            }
            while ($statement->fetch()) {
                continue;
            }
            $pool->put($mysqli);
        });
    }
});
$s = microtime(true) - $s;
echo '쿼리 ' . N . '개에 ' . $s . '초 소요됐습니다' . PHP_EOL;
```
