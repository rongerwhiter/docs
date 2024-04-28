＃ Coroutine\Scheduler

?> All [coroutines](/coroutine) must be [created](/coroutine/coroutine?id=create) within a `coroutine container`. In most cases, when a `Swoole` program starts, it will automatically create a `coroutine container`. There are three ways to start a program using `Swoole`:

   - Call the [start](/server/methods?id=start) method for starting a server program in [asynchronous style](/server/init). This method of starting will create a `coroutine container` in the event callback, see [enable_coroutine](/server/setting?id=enable_coroutine) for reference.
   - Call the [start](/process/process_pool?id=start) method of the two process management modules provided by `Swoole`, [Process](/process/process) and [Process\Pool](/process/process_pool). This method of starting will create a `coroutine container` when the process starts. See the `enable_coroutine` parameter in the constructors of these two modules for reference.
   - Other ways to start a program directly by writing coroutines require creating a coroutine container first (`Coroutine\run()` function, which can be understood as the `main` function in Java or C), for example:

* **Start a full coroutine `HTTP` service**

```php
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/', function ($request, $response) {
        $response->end("<h1>Index</h1>");
    });
    $server->handle('/test', function ($request, $response) {
        $response->end("<h1>Test</h1>");
    });
    $server->handle('/stop', function ($request, $response) use ($server) {
        $response->end("<h1>Stop</h1>");
        $server->shutdown();
    });
    $server->start();
});
echo 1;// Will not be executed
```

* **Add 2 concurrent coroutines to do something**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    Coroutine::create(function() {
        var_dump(file_get_contents("http://www.xinhuanet.com/"));
    });

    Coroutine::create(function() {
        Coroutine::sleep(1);
        echo "done\n";
    });
});
echo 1;// Will be executed
```

!> Available in `Swoole v4.4+`.

!> `Coroutine\run()` cannot be nested.  
If there are events that have not been processed within `Coroutine\run()`, the code after `Coroutine\run()` will not be executed. Conversely, if there are no events, the code will continue to run downwards, and `Coroutine\run()` can be called again.

The `Coroutine\run()` function mentioned above is actually a wrapper for the `Swoole\Coroutine\Scheduler` class (coroutine scheduler class). If you want to know more details, you can refer to the methods of `Swoole\Coroutine\Scheduler`:
### set()

?> **Set runtime parameters for the coroutine.**

?> This is an alias of `Coroutine::set` method. Please refer to the [Coroutine::set](/coroutine/coroutine?id=set) documentation.

```php
Swoole\Coroutine\Scheduler->set(array $options): bool
```

  * **Example**

```php
$sch = new Swoole\Coroutine\Scheduler;
$sch->set(['max_coroutine' => 100]);
```
### getOptions()

?> **Retrieve the set coroutine runtime parameters.** Available since Swoole version `v4.6.0`

?> This is an alias of the `Coroutine::getOptions` method. Please refer to the [Coroutine::getOptions](/coroutine/coroutine?id=getoptions) documentation

```php
Swoole\Coroutine\Scheduler->getOptions(): null|array
```
### add()

?> **タスクを追加します。**

```php
Swoole\Coroutine\Scheduler->add(callable $fn, ... $args): bool
```

  * **パラメータ** 

    * **`callable $fn`**
      * **機能**：コールバック関数
      * **デフォルト値**：なし
      * **他の値**：なし

    * **`... $args`**
      * **機能**：オプションのパラメータ、コルーチンに渡されます
      * **デフォルト値**：なし
      * **他の値**：なし

  * **例**

```php
use Swoole\Coroutine;

$scheduler = new Coroutine\Scheduler;
$scheduler->add(function ($a, $b) {
    Coroutine::sleep(1);
    echo assert($a == 'hello') . PHP_EOL;
    echo assert($b == 12345) . PHP_EOL;
    echo "Done.\n";
}, "hello", 12345);

$scheduler->start();
```

  * **注意**

    !> `go` 関数とは異なり、ここで追加されたコルーチンは即座に実行されず、`start` メソッドが呼び出されるまで待機し、一緒に起動および実行されます。プログラムで単にコルーチンを追加し、`start` を呼び出さずに起動しない場合、コルーチン関数 `$fn` は実行されません。
### parallel()

?> **Add parallel tasks.**

?> Unlike the `add` method, the `parallel` method creates parallel coroutines. When `start` is called, `$num` and `$fn` coroutines are started simultaneously and executed in parallel.

```php
Swoole\Coroutine\Scheduler->parallel(int $num, callable $fn, ... $args): bool
```

  * **Parameters**

    * **`int $num`**
      * **Description**: Number of coroutines to start
      * **Default**: None
      * **Other values**: None

    * **`callable $fn`**
      * **Description**: Callback function
      * **Default**: None
      * **Other values**: None

    * **`... $args`**
      * **Description**: Optional parameters passed to the coroutine
      * **Default**: None
      * **Other values**: None

  * **Example**

```php
use Swoole\Coroutine;

$scheduler = new Coroutine\Scheduler;

$scheduler->parallel(10, function ($t, $n) {
    Coroutine::sleep($t);
    echo "Co ".Coroutine::getCid()."\n";
}, 0.05, 'A');

$scheduler->start();
```  
### start()

?> **Start the program.** 

?> Traverse through the coroutine tasks added by `add` and `parallel` methods and execute them.

```php
Swoole\Coroutine\Scheduler->start(): bool
```

  * **Return Value**

    * If started successfully, all added tasks will be executed, and when all coroutines exit, `start` will return `true`.
    * If failed to start, it returns `false`. The reason could be that it is already started or that another scheduler has been created and it cannot be created again.
