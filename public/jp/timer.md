# タイマー Timer

ミリ秒単位の精度を持つタイマーです。内部では、`epoll_wait`および`setitimer`を使用し、データ構造には`最小ヒープ`が使われており、大量のタイマーの追加をサポートしています。

- `Manager`や`TaskWorker`プロセスなどの同期IOプロセスでは、`setitimer`とシグナルを使用して実装されています。
- 非同期IOプロセスでは、`epoll_wait`/`kevent`/`poll`/`select`のタイムアウト時間を使用して実装されています。
## パフォーマンス

最小ヒープデータ構造を使用してタイマーを実装し、タイマーの追加と削除はすべてメモリ操作なので、パフォーマンスは非常に高いです。

> 公式のベンチマークスクリプト [timer.php](https://github.com/swoole/benchmark/blob/master/timer.php) では、ランダムなタイマー`10`万件を追加または削除するのに約`0.08秒`かかります。

```shell
~/workspace/swoole/benchmark$ php timer.php
add 100000 timer :0.091133117675781s
del 100000 timer :0.084658145904541s
```

!> タイマーはメモリ操作であり、`IO`コストはかかりません。
## 差異

`Timer`と`PHP`の`pcntl_alarm`は異なります。`pcntl_alarm`は`クロック信号 + tick`関数に基づいているため、いくつかの欠点があります：

  * ミリ秒単位までサポートできる`Timer`に対して、最大で秒までしかサポートできない
  * 複数のタイマープログラムを同時に設定できない
  * `pcntl_alarm`は`declare(ticks = 1)`に依存しており、パフォーマンスが低い
## ゼロミリ秒タイマー

ゼロを設定することはサポートされていないため、低レベルでは時間パラメーターが`0`のタイマーがサポートされていません。これは`Node.js`などのプログラミング言語とは異なります。`Swoole`では、[Swoole\Event::defer](/event?id=defer)を使用して同様の機能を実現することができます。

```php
Swoole\Event::defer(function () {
  echo "hello\n";
});
```

!> 上記のコードは、`JS`の`setTimeout(func, 0)`と同等の効果を持っています。
## 別名

`tick()`、`after()`、`clear()` はすべて関数スタイルの別名を持っています。

クラスの静的なメソッド | 関数スタイルの別名
---|---
`Swoole\Timer::tick()` | `swoole_timer_tick()`
`Swoole\Timer::after()` | `swoole_timer_after()`
`Swoole\Timer::clear()` | `swoole_timer_clear()`
This is already in Japanese. Is there anything else you needed help with?
### tick()

間隔時計タイマーを設定します。

`after`タイマーとは異なり、`tick`タイマーは[Tips::clear](/timer?id=clear)を呼び出すまで継続的にトリガーされます。

```php
Swoole\Timer::tick(int $msec, callable $callback_function, ...$params): int
```

!> 1. タイマーは現在のプロセス空間でのみ有効です
   2. タイマーは純粋な非同期実装であり、[同期I/O](/learn?id=同期io必須)関数と一緒に使用できません。さもないと、タイマーの実行時間が乱れる可能性があります
   3. タイマーの実行中にわずかな誤差が発生する可能性があります

  * **Parameters** 

    * **`int $msec`**
      * **説明**：指定された時間
      * **単位**：ミリ秒【例：`1000`は`1`秒を表し、`v4.2.10`未満のバージョンでは最大で `86400000`を超えることはできません】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`callable $callback_function`**
      * **説明**：時間が経過した後に実行される関数、呼び出し可能である必要があります
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`...$params`**
      * **説明**：実行関数にデータを渡す【このパラメータもオプションです】
      * **デフォルト値**：なし
      * **その他の値**：なし
      
      !> コールバック関数にパラメーターを渡すために匿名関数の`use`構文を使用できます

  * **$callback_function コールバック関数** 

    ```php
    callbackFunction(int $timer_id, ...$params);
    ```

      * **`int $timer_id`**
        * **説明**：タイマーの`ID`【[Timer::clear](/timer?id=clear)でこのタイマーをクリアするために使用できます】
        * **デフォルト値**：なし
        * **その他の値**：なし

      * **`...$params`**
        * **説明**：`Timer::tick`から渡された第三引数`$param`
        * **デフォルト値**：なし
        * **その他の値**：なし

  * **Extensions**

    * **タイマーの修正**

      タイマーコールバック関数の実行時間は次回のタイマー実行時間に影響を与えません。例：`0.002s`で`10ms`の`tick`タイマーを設定し、最初は`0.012s`にコールバック関数が実行されます。コールバック関数が`5ms`を実行した場合でも、次のタイマーは`0.022s`にトリガーされますが、`0.027s`ではありません。
      
      ただし、タイマーコールバック関数の実行時間が長すぎて、次回のタイマー実行時刻を網羅する場合があります。その場合、ベース層は時間の補正を行い、期限切れの行動を破棄し、次の時間にコールバックします。上記の例では`0.012s`でのコールバック関数が`15ms`を実行し、本来は`0.022s`にタイミングコールバックが発生するはずでした。実際にはこのタイマーは`0.027s`に返されますが、その時点でタイミングは既に過ぎています。ベース層は`0.032s`で再度タイマーコールバックをトリガーします。

    * **コルーチンモード**

      コルーチン環境では、`Timer::tick`コールバック内で自動的にコルーチンが作成され、コルーチン関連の`API`を直接使用でき、`go`を呼び出す必要はありません。
      
      !> [enable_coroutine](/timer?id=close-timer-co)を設定して自動的にコルーチン作成を無効にすることができます

  * **使用例**

    ```php
    Swoole\Timer::tick(1000, function(){
        echo "timeout\n";
    });
    ```

    * **正しい例**

    ```php
    Swoole\Timer::tick(3000, function (int $timer_id, $param1, $param2) {
        echo "timer_id #$timer_id, after 3000ms.\n";
        echo "param1 is $param1, param2 is $param2.\n";

        Swoole\Timer::tick(14000, function ($timer_id) {
            echo "timer_id #$timer_id, after 14000ms.\n";
        });
    }, "A", "B");
    ```

    * **間違った例**

    ```php
    Swoole\Timer::tick(3000, function () {
        echo "after 3000ms.\n";
        sleep(14);
        echo "after 14000ms.\n";
    });
    ```
### after()

指定された時間の後に関数を実行します。`Swoole\Timer::after`関数は一度だけ実行され、完了すると破棄されます。

この関数は`PHP`標準ライブラリの`sleep`関数とは異なり、`after`は非ブロッキングです。一方、`sleep`を呼び出すと現在のプロセスがブロックされ、新しいリクエストを処理できなくなります。

```php
Swoole\Timer::after(int $msec, callable $callback_function, ...$params): int
```

  * **Parameters** 

    * **`int $msec`**
      * **Description**: Specifies the time
      * **Unit**: milliseconds【e.g. `1000` indicates `1` second, with a maximum of `86400000` for versions below `v4.2.10`】
      * **Default**: N/A
      * **Other values**: N/A

    * **`callable $callback_function`**
      * **Description**: The function to be executed after the specified time, must be callable.
      * **Default**: N/A
      * **Other values**: N/A

    * **`...$params`**
      * **Description**: Pass data to the executed function【This parameter is optional】
      * **Default**: N/A
      * **Other values**: N/A
      
      !> You can pass arguments to the callback function using the use syntax of anonymous functions

  * **Return Value**

    * Returns the timer `ID` if successful, to cancel the timer, you can call [Swoole\Timer::clear](/timer?id=clear)

  * **Extensions**

    * **Coroutine Mode**

      In a coroutine environment, a coroutine will be automatically created in the [Swoole\Timer::after](/timer?id=after) callback, allowing you to use coroutine-related APIs directly without having to call `go` to create a coroutine.
      
      !> You can set [enable_coroutine](/timer?id=close-timer-co) to disable the automatic creation of coroutines

  * **Example**

```php
$str = "Swoole";
Swoole\Timer::after(1000, function() use ($str) {
    echo "Hello, $str\n";
});
```
```php
Swoole\Timer::clear(int $timer_id): bool
```

  * **パラメータ** 

    * **`int $timer_id`**
      * **機能**：タイマー`ID`【[Timer::tick](/timer?id=tick)、[Timer::after](/timer?id=after)を呼び出した後に返される整数のID】
      * **デフォルト値**：なし
      * **その他の値**：なし

!> `Swoole\Timer::clear`は他のプロセスのタイマーを削除するために使用できません。現在のプロセスにのみ有効です。

  * **使用例**

```php
$timer = Swoole\Timer::after(1000, function () {
    echo "timeout\n";
});

var_dump(Swoole\Timer::clear($timer));
var_dump($timer);

// 出力：bool(true) int(1)
// 出力しない：timeout
```
### clearAll()

すべてのタイマーをクリアします。

!> Swooleバージョン >= `v4.4.0`で利用可能

```php
Swoole\Timer::clearAll(): bool
```
### info()

`timer`の情報を返します。

!> Swooleバージョン >= `v4.4.0`で利用可能

```php
Swoole\Timer::info(int $timer_id): array
```

  * **Return Value**

```php
array(5) {
  ["exec_msec"]=>
  int(6000)
  ["exec_count"]=> // 追加v4.8.0
  int(5)
  ["interval"]=>
  int(1000)
  ["round"]=>
  int(0)
  ["removed"]=>
  bool(false)
}
```
### list()

定時器のイテレーターを返します。これを使用して、現在のWorkerプロセス内のすべての`timer`のIDを`foreach`で繰り返すことができます。

!> Swooleバージョン >= `v4.4.0` で利用可能

```php
Swoole\Timer::list(): Swoole\Timer\Iterator
```

* **使用例**

```php
foreach (Swoole\Timer::list() as $timer_id) {
    var_dump(Swoole\Timer::info($timer_id));
}
```
### stats()

定時器の状態を確認します。

!> Swooleバージョン >= `v4.4.0` で利用可能

```php
Swoole\Timer::stats(): array
```

  * **戻り値**

```php
array(3) {
  ["initialized"]=>
  bool(true)
  ["num"]=>
  int(1000)
  ["round"]=>
  int(1)
}
```  
### set()

設定タイマー関連のパラメータ。

```php
Swoole\Timer::set(array $array): void
```

!> このメソッドは `v4.6.0` 以降、非推奨となりました。
## 协程の終了 :id=close-timer-co

デフォルトでは、タイマーはコールバック関数の実行時に自動的にコルーチンを作成しますが、タイマーのコルーチンを個別に終了することもできます。

```php
swoole_async_set([
  'enable_coroutine' => false,
]);
```
