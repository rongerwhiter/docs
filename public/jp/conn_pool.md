# 接続プール

Swooleは`v4.4.13`バージョンから組み込みのコルーチン接続プールを提供しています。このセクションでは、対応する接続プールの使用方法について説明します。
## ConnectionPool

[ConnectionPool](https://github.com/swoole/library/blob/master/src/core/ConnectionPool.php)は、基本的な接続プールであり、Channelをベースに自動スケジューリングを行います。任意の構築子（`callable`）を指定でき、構築子は接続オブジェクトを返す必要があります。

* `get`メソッドは接続を取得します（接続プールがいっぱいの場合は新しい接続が作成されます）。
* `put`メソッドは接続を回収します。
* `fill`メソッドは接続プールを満たします（事前に接続を作成します）。
* `close`メソッドは接続プールを閉じます。

!> [Simps Framework](https://simps.io) の [DBコンポーネント](https://github.com/simple-swoole/db) は、Databaseをラップしており、自動的に接続を返却したり、トランザクションなどの機能を実現しています。これを参考にするか、直接使用することができます。詳細は[Simpsドキュメント](https://simps.io/#/zh-cn/database/mysql)をご覧ください。
## データベース

さまざまなデータベース接続プールとオブジェクトプロキシの高度なラッピングで、自動再接続がサポートされています。現在、PDO、Mysqli、Redisの3種類のデータベースサポートが含まれています：

* `PDOConfig`, `PDOProxy`, `PDOPool`
* `MysqliConfig`, `MysqliProxy`, `MysqliPool`
* `RedisConfig`, `RedisProxy`, `RedisPool`

!> 1. MySQLの再接続により、ほとんどの接続コンテキスト（フェッチモード、設定された属性、コンパイル済みのステートメントなど）は自動的に復元されますが、トランザクションなどのコンテキストは復元されません。トランザクション中の接続が切断された場合、例外がスローされますので、再接続の信頼性を自己評価してください；  
2. トランザクション中の接続をプールに返却することは定義されていない動作です。開発者は再利用可能な接続が返却されることを自己保証する必要があります；  
3. 再利用できない接続がある場合、開発者は `$pool->put(null);` を呼び出して空の接続を返却し、接続プールの数を適切に保つ必要があります。
### PDOPool/MysqliPool/RedisPool :id=pool

接続プールオブジェクトを作成するためのパラメータが2つあります。それぞれ、対応するConfigオブジェクトとプールのサイズです。

```php
$pool = new \Swoole\Database\PDOPool(Swoole\Database\PDOConfig $config, int $size);

$pool = new \Swoole\Database\MysqliPool(Swoole\Database\MysqliConfig $config, int $size);

$pool = new \Swoole\Database\RedisPool(Swoole\Database\RedisConfig $config, int $size);
```

  * **パラメータ** 

    * **`$config`**
      * **目的**：対応するConfigオブジェクト、具体的な使用方法は後述の[Usage Examples](/coroutine/conn_pool?id=使用例)を参照してください
      * **デフォルト値**：なし
      * **他の値**：【[PDOConfig](https://github.com/swoole/library/blob/master/src/core/Database/PDOConfig.php)、[RedisConfig](https://github.com/swoole/library/blob/master/src/core/Database/RedisConfig.php)、[MysqliConfig](https://github.com/swoole/library/blob/master/src/core/Database/MysqliConfig.php)】
      
    * **`int $size`**
      * **目的**：接続プールのサイズ
      * **デフォルト値**：64
      * **他の値**：なし
Here is an example of how you can use this feature:

```
print("Hello, World!")
```

Feel free to modify the code to suit your needs. 

## お役に立てれば幸いです。
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
                throw new RuntimeException('Prepare failed');
            }
            $a = mt_rand(1, 100);
            $b = mt_rand(1, 100);
            $result = $statement->execute([$a, $b]);
            if (!$result) {
                throw new RuntimeException('Execute failed');
            }
            $result = $statement->fetchAll();
            if ($a + $b !== (int)$result[0][0]) {
                throw new RuntimeException('Bad result');
            }
            $pool->put($pdo);
        });
    }
});
$s = microtime(true) - $s;
echo 'Use ' . $s . 's for ' . N . ' queries' . PHP_EOL;
```
### Redis

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
                throw new RuntimeException('Prepare failed');
            }
            $a = mt_rand(1, 100);
            $b = mt_rand(1, 100);
            if (!$statement->bind_param('dd', $a, $b)) {
                throw new RuntimeException('Bind param failed');
            }
            if (!$statement->execute()) {
                throw new RuntimeException('Execute failed');
            }
            if (!$statement->bind_result($result)) {
                throw new RuntimeException('Bind result failed');
            }
            if (!$statement->fetch()) {
                throw new RuntimeException('Fetch failed');
            }
            if ($a + $b !== (int)$result) {
                throw new RuntimeException('Bad result');
            }
            while ($statement->fetch()) {
                continue;
            }
            $pool->put($mysqli);
        });
    }
});
$s = microtime(true) - $s;
echo 'Use ' . $s . 's for ' . N . ' queries' . PHP_EOL;
```
