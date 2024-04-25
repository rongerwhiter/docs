# Swoole\Thread

`6.0` バージョンからマルチスレッドサポートが提供され、スレッドAPIを使用してマルチプロセスの代わりに使用できます。複数のプロセスと比較して、`Thread` はより豊富な並行データコンテナを提供し、ゲームサーバーや通信サーバーの開発に便利です。

## コンパイル
- `PHP` は `ZTS` モードである必要があり、`--enable-zts` オプションを使用して`PHP` をコンパイルする必要があります。
- `Swoole` をコンパイルする際には、`--enable-swoole-thread` コンパイルオプションを追加する必要があります。

`--enable-swoole-thread` コンパイルオプションは非スレッドモードと排他的であり、`Server`、`Atomic`、`Lock` モジュールは`thread`機能を有効にした後はマルチプロセスモードでは使用できません。

## 情報を確認する

```shell
php -v
PHP 8.1.23 (cli) (built: Mar 20 2024 19:48:19) (ZTS)
Copyright (c) The PHP Group
Zend Engine v4.1.23, Copyright (c) Zend Technologies
```

`(ZTS)` はスレッドセーフが有効であることを示します。

```shell
php --ri swoole
php --ri swoole

swoole

Swoole => enabled
...
thread => enabled
...
```

`thread => enabled` はマルチスレッドサポートが有効であることを示します。

## スレッドの作成

```php
Thread::exec(string $script_file, array ...$argv);
```

注意: 作成された子スレッドは親スレッドからの資源継承を行わないため、子スレッド内では以下の内容がクリアされ、再作成または設定が必要です：
- 読み込まれた`PHP`ファイルを再度`include/require`する必要があります
- `autoload`
- クラス、関数、定数はクリアされ、`PHP`ファイルを再読み込みして作成する必要があります
- グローバル変数、例:`$GLOBALS`、`$_GET/$_POST`など、クリアされます
- クラスの静的プロパティ、関数の静的変数は初期値にリセットされます
- `php.ini` のオプション、例:`error_reporting()`は子スレッド内で再設定する必要があります

データを子スレッドに渡すにはスレッドパラメータを使用する必要があります。子スレッド内でも新しいスレッドを作成することができます。

### パラメータ
- `$script_file`: スレッドを起動した後に実行するスクリプト
- `...$argv`: スレッドパラメータを渡します。シリアライズ可能な変数である必要があり、`resource`リソースハンドルは渡すことができず、子スレッド内で`Thread::getArguments()`を使用して取得できます。

### 戻り値
`Thread` オブジェクトを返し、親スレッド内で子スレッドを`join()`などで制御することができます。

スレッドオブジェクトが破棄されると自動的に`join()`が呼ばれて子スレッドの終了を待ちます。これによりブロッキングが発生する可能性がありますが`$thread->detach()`メソッドを使用して子スレッドを親から切り離して独立して実行させることができます。

### 例
```php
use Swoole\Thread;

$args = Thread::getArguments();
$c = 4;

if (empty($args)) {
    # 親スレッド
    for ($i = 0; $i < $c; $i++) {
        $threads[] = Thread::exec(__FILE__, $i);
    }
    for ($i = 0; $i < $c; $i++) {
        $threads[$i]->join();
    }
} else {
    # 子スレッド
    echo "Thread #" . $args[0] . "\n";
    while (1) {
        sleep(1);
        file_get_contents('https://www.baidu.com/');
    }
}
```

## 定数
- `Thread::HARDWARE_CONCURRENCY` ハードウェアシステムがサポートする並行スレッド数、つまり`CPU`コア数を取得します

## Server
- すべてのワーカープロセスはスレッドを使用して実行され、`Worker`、`Task Worker`、`User Process`を含みます
- `bootstrap` と `init_arguments` の2つの設定が追加され、ワーカーのエントリースクリプトファイルとスレッドの初期化パラメータを設定するために使用されます

```php
$http = new Swoole\Http\Server("0.0.0.0", 9503);
$http->set([
    'worker_num' => 2,
    'task_worker_num' => 3,
    'bootstrap' => __FILE__,
    'init_arguments' => function () use ($http) {
        $map = new Swoole\Thread\Map;
        return [$map];
    }
]);

$http->on('Request', function ($req, $resp) use ($http) {
    $resp->end('hello world');
});

$http->on('pipeMessage', function ($http, $srcWorkerId, $msg) {
    echo "[worker#" . $http->getWorkerId() . "]\treceived pipe message[$msg] from " . $srcWorkerId . "\n";
});

$http->addProcess(new \Swoole\Process(function () {
   echo "user process, id=" . \Swoole\Thread::getId();
   sleep(2000);
}));

$http->on('Task', function ($server, $taskId, $srcWorkerId, $data) {
    var_dump($taskId, $srcWorkerId, $data);
    return ['result' => uniqid()];
});

$http->on('Finish', function ($server, $taskId, $data) {
    var_dump($taskId, $data);
});

$http->on('WorkerStart', function ($serv, $wid) {
    var_dump(\Swoole\Thread::getArguments(), $wid);
});

$http->on('WorkerStop', function ($serv, $wid) {
    var_dump('stop: T' . \Swoole\Thread::getId());
});

$http->start();
```
