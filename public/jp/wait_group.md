# Coroutine\WaitGroup

`Swoole4`では、[Channel](/coroutine/channel)を使用して、コルーチン間の通信、依存管理、およびコルーチンの同期を実現できます。[Channel](/coroutine/channel)をベースにして、`Golang`の`sync.WaitGroup`機能を簡単に実装できます。

## 実装コード

> この機能はPHPで記述されたものであり、C/C++コードではありません。実装元のソースコードは [Library](https://github.com/swoole/library/blob/master/src/core/Coroutine/WaitGroup.php) にあります。

* `add`メソッドはカウントを増やします
* `done`はタスクが完了したことを示します
* `wait`はすべてのタスクが完了するのを待ち、現在のコルーチンの実行を再開します
* `WaitGroup`オブジェクトは再利用でき、`add`、`done`、`wait`の後に再度使用できます

## 使用例

```php
<?php
use Swoole\Coroutine;
use Swoole\Coroutine\WaitGroup;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $wg = new WaitGroup();
    $result = [];

    $wg->add();
    // First coroutine
    Coroutine::create(function () use ($wg, &$result) {
        // Start a coroutine client, request Taobao homepage
        $cli = new Client('www.taobao.com', 443, true);
        $cli->setHeaders([
            'Host' => 'www.taobao.com',
            'User-Agent' => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $cli->set(['timeout' => 1]);
        $cli->get('/index.php');

        $result['taobao'] = $cli->body;
        $cli->close();

        $wg->done();
    });

    $wg->add();
    // Second coroutine
    Coroutine::create(function () use ($wg, &$result) {
        // Start a coroutine client, request Baidu homepage
        $cli = new Client('www.baidu.com', 443, true);
        $cli->setHeaders([
            'Host' => 'www.baidu.com',
            'User-Agent' => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $cli->set(['timeout' => 1]);
        $cli->get('/index.php');

        $result['baidu'] = $cli->body;
        $cli->close();

        $wg->done();
    });

    // Suspend the current coroutine and wait for all tasks to complete before resuming
    $wg->wait();
    // Here $result contains the results of 2 tasks
    var_dump($result);
});
```
