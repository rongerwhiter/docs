# 配置

[Swoole\Server->set()](/server/methods?id=set) 関数は、`Server`が実行される際の各種パラメータを設定するために使用されます。このセクションのすべてのサブページは、設定配列の要素です。

!> [v4.5.5](/version/log?id=v455) バージョン以降、Swooleは設定された設定項目をチェックし、Swooleが提供しない設定項目が設定されている場合は警告が発生します。

```shell
PHP Warning:  unsupported option [foo] in @swoole-src/library/core/Server/Helper.php 
```
### debug_mode

?> Set the log mode to `debug` for debugging purposes, only effective if `--enable-debug` is enabled during compilation.

```php
$server->set([
  'debug_mode' => true
])
```
### トレースフラグ

?> トレースログのタグを設定し、一部のトレースログのみを出力します。`trace_flags` は複数のトレース項目を設定するために `|` 論理和演算子をサポートしています。`--enable-trace-log` がコンパイル時に有効にされている場合にのみ機能します。

以下はサポートされているトレース項目です。`SWOOLE_TRACE_ALL` を使用してすべての項目をトレースすることができます：

* `SWOOLE_TRACE_SERVER`
* `SWOOLE_TRACE_CLIENT`
* `SWOOLE_TRACE_BUFFER`
* `SWOOLE_TRACE_CONN`
* `SWOOLE_TRACE_EVENT`
* `SWOOLE_TRACE_WORKER`
* `SWOOLE_TRACE_REACTOR`
* `SWOOLE_TRACE_PHP`
* `SWOOLE_TRACE_HTTP2`
* `SWOOLE_TRACE_EOF_PROTOCOL`
* `SWOOLE_TRACE_LENGTH_PROTOCOL`
* `SWOOLE_TRACE_CLOSE`
* `SWOOLE_TRACE_HTTP_CLIENT`
* `SWOOLE_TRACE_COROUTINE`
* `SWOOLE_TRACE_REDIS_CLIENT`
* `SWOOLE_TRACE_MYSQL_CLIENT`
* `SWOOLE_TRACE_AIO`
* `SWOOLE_TRACE_ALL`
### log_file

**Swoole error log file**

Swoole内で発生した例外情報は、デフォルトではこのファイルに記録され、通常は画面に出力されます。  
デーモンモードが有効になっている場合（`daemonize => true`）、標準出力は`log_file`にリダイレクトされます。PHPコード中の `echo/var_dump/print` などの画面に出力される内容は `log_file` ファイルに書き込まれます。

* **ヒント**

  * `log_file`に記録されたログは実行時のエラー記録にすぎず、長期的な保存の必要はありません。

  * **ログ識別子**

    ログ情報には、プロセスIDの前にいくつかの識別子が付けられ、ログの発生元のスレッド/プロセスの種類を示します。

      * `#` マスタープロセス
      * `$` マネージャープロセス
      * `*` ワーカープロセス
      * `^` タスクプロセス

  * **ログファイルを再オープン**

    サーバープログラムが実行中に、ログファイルが`mv`で移動されたり`unlink`で削除された場合、ログ情報が正常に書き込まれなくなります。この場合、`Server`に`SIGRTMIN`シグナルを送信することで、ログファイルを再度オープンできます。

    * Linux プラットフォームのみサポートされています。
    * [UserProcess](/server/methods?id=addProcess)プロセスはサポートされていません。

* **注意**

    `log_file`はファイルを自動的に分割しないため、定期的にこのファイルをクリーンアップする必要があります。`log_file`の出力を観察することで、サーバーの各種異常情報や警告を得ることができます。
### log_level

?> **`Server`エラーログのレベルを設定します。範囲は`0-6`です。`log_level`で設定されたレベルより低いログ情報は表示されません。**【デフォルト値：`SWOOLE_LOG_INFO`】

対応するレベルの定数は[ログレベル](/consts?id=ログレベル)を参照してください。

* **注意**

    !> `SWOOLE_LOG_DEBUG`および`SWOOLE_LOG_TRACE`は[--enable-debug-log](/environment?id=debugパラメータ)および[--enable-trace-log](/environment?id=debugパラメータ)バージョンでのみ使用可能です。  
    `daemonize`デーモンプロセスを有効にすると、プログラム内のすべてのコンソール出力が[log_file](/server/setting?id=log_file)に書き込まれます。この部分は`log_level`によって制御されません。
### log_date_format

?> **`Server`のログ時間形式を設定**します。形式は [strftime](https://www.php.net/manual/zh/function.strftime.php) の`format`を参照してください。

```php
$server->set([
    'log_date_format' => '%Y-%m-%d %H:%M:%S',
]);
```
### log_date_with_microseconds

?> **Set the `Server` log precision with microseconds.** 【Default Value: `false`】
### ログ回転

?> **`Server`のログの分割を設定します**【デフォルト値：`SWOOLE_LOG_ROTATION_SINGLE`】

| 定数                             | 説明   | バージョン情報 |
| -------------------------------- | ------ | -------------- |
| SWOOLE_LOG_ROTATION_SINGLE       | 無効   | -              |
| SWOOLE_LOG_ROTATION_MONTHLY      | 月毎   | v4.5.8         |
| SWOOLE_LOG_ROTATION_DAILY        | 日毎   | v4.5.2         |
| SWOOLE_LOG_ROTATION_HOURLY       | 時毎   | v4.5.8         |
| SWOOLE_LOG_ROTATION_EVERY_MINUTE | 分毎   | v4.5.8         |
### display_errors

?> `Swoole` エラーメッセージの表示を有効 / 無効にします。

```php
$server->set([
  'display_errors' => true
])
```
### dns_server

?> Set the IP address for DNS queries.
### socket_dns_timeout

?> The domain name resolution timeout, if coroutine client is enabled on the server side, this parameter can control the domain name resolution timeout for the client, in seconds.
### socket_connect_timeout

?> The client connection timeout, if the server uses coroutine clients, this parameter can control the client's connection timeout in seconds.
### socket_write_timeout / socket_send_timeout

?> The client write timeout sets the timeout for writing by the client. If the server enables coroutines for clients, this parameter can control the client's write timeout in seconds.  
This configuration can also be used to control the execution timeout for `shell_exec` after enabling coroutines or [Swoole\Coroutine\System::exec()](/coroutine/system?id=exec).
### socket_read_timeout / socket_recv_timeout

?> クライアントのリードタイムアウト、サーバーでコルーチンクライアントを有効にしている場合、このパラメータはクライアントのリードタイムアウトを制御します。単位は秒です。
### max_coroutine / max_coro_num：id=max_coroutine

?> **現在のワーカープロセスの最大コルーチン数を設定します。**【デフォルト値：`100000`、Swooleバージョンが`v4.4.0-beta`より前の場合はデフォルト値は`3000`です】

?> `max_coroutine`を超えると、下位レイヤーが新しいコルーチンを作成できなくなり、Swooleのサーバーは`exceed max number of coroutine`エラーをスローし、`TCP Server`は接続を直接閉じ、`Http Server`はHttpの503ステータスコードを返します。

?> `Server`プログラムで実際に作成できる最大コルーチン数は`worker_num * max_coroutine`に等しくなります。taskプロセスおよびUserProcessプロセスのコルーチン数はそれぞれ別途計算されます。

```php
$server->set(array(
    'max_coroutine' => 3000,
));
```
### enable_deadlock_check

?> Coroutine deadlock detection is enabled.

```php
$server->set([
  'enable_deadlock_check' => true
]);
```  
### hook_flags

?> **Set the function scope for `one-click coroutine` hooking.**【Default: no hook】

!> Available in Swoole version `v4.5+` or [4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x), for more details see [One-click Coroutine](/runtime)

```php
$server->set([
    'hook_flags' => SWOOLE_HOOK_SLEEP,
]);
```

The following coroutine items are supported at the underlying level, `SWOOLE_HOOK_ALL` can be used to represent coroutine all:

* `SWOOLE_HOOK_TCP`
* `SWOOLE_HOOK_UNIX`
* `SWOOLE_HOOK_UDP`
* `SWOOLE_HOOK_UDG`
* `SWOOLE_HOOK_SSL`
* `SWOOLE_HOOK_TLS`
* `SWOOLE_HOOK_SLEEP`
* `SWOOLE_HOOK_FILE`
* `SWOOLE_HOOK_STREAM_FUNCTION`
* `SWOOLE_HOOK_BLOCKING_FUNCTION`
* `SWOOLE_HOOK_PROC`
* `SWOOLE_HOOK_CURL`
* `SWOOLE_HOOK_NATIVE_CURL`
* `SWOOLE_HOOK_SOCKETS`
* `SWOOLE_HOOK_STDIO`
* `SWOOLE_HOOK_PDO_PGSQL`
* `SWOOLE_HOOK_PDO_ODBC`
* `SWOOLE_HOOK_PDO_ORACLE`
* `SWOOLE_HOOK_PDO_SQLITE`
* `SWOOLE_HOOK_ALL`
### enable_preemptive_scheduler

?> Setting to enable pre-emptive scheduling of coroutines to prevent one coroutine from starving others due to long execution time. The maximum execution time for a coroutine is `10ms`.

```php
$server->set([
  'enable_preemptive_scheduler' => true
]);
```
### c_stack_size / stack_size

?> Set the memory size of the initial C stack of a single coroutine, which is 2M by default.
### aio_core_worker_num

?> Set the minimum number of `AIO` working threads, with the default value being the number of CPU cores.
### aio_worker_num 

?> The maximum number of working threads for `AIO` is set, and the default value is `cpu` cores * 8.
### aio_max_wait_time

?> タスクスレッドが待機する最大時間、単位は秒です。
### aio_max_idle_time

?> The maximum idle time of the worker threads, in seconds.
### reactor_num

?> **Set the number of [Reactor](/learn?id=reactor-thread) threads to start.**【Default value: the number of `CPU` cores】

?> Use this parameter to adjust the number of event processing threads within the main process to fully utilize multiple cores. By default, the same number of threads as the number of `CPU` cores will be started.  
Reactor threads can utilize multiple cores, for example: if the machine has `128` cores, then `128` threads will be started at the low level.  
Each thread will maintain an [EventLoop](/learn?id=what-is-eventloop). The threads are lock-free, and instructions can be executed in parallel by the `128` CPU cores.  
Considering the performance loss from operating system scheduling, it is recommended to set it to CPU cores * 2 in order to maximize the utilization of each CPU core.

  * **Hint**

    * It is recommended to set `reactor_num` to `1-4` times the number of `CPU` cores.
    * `reactor_num` must not exceed [swoole_cpu_num()](/functions?id=swoole_cpu_num) * 4

  * **Caution**

  !> - `reactor_num` must be less than or equal to `worker_num`;
- If `reactor_num` is set higher than `worker_num`, it will automatically adjust to make `reactor_num` equal to `worker_num`;
- On machines with more than `8` cores, `reactor_num` defaults to `8`.
### worker_num

?> **Set the number of `Worker` processes to start.**【Default value: the number of `CPU` cores】

?> If one request takes `100ms` to process and you need to provide processing capacity of `1000QPS`, you must configure `100` or more processes.  
However, the more processes you start, the more memory will be consumed significantly, and the overhead of context switching between processes will also increase. So, configure it appropriately here. Do not set it too large.

  * **Tips**

    * If the business logic is fully [asynchronous IO](https://www.example.com/learn?lang=en&id=sync-async-io), setting this to `1-4` times the number of `CPU` cores is most reasonable.
    * If the business logic involves [synchronous IO](https://www.example.com/learn?lang=en&id=sync-async-io), you need to adjust it based on the request response time and system load, for example: `100-500`.
    * It is set by default to [swoole_cpu_num()](https://www.example.com/functions?lang=en&id=swoole_cpu_num), and the maximum value should not exceed [swoole_cpu_num()](https://www.example.com/functions?lang=en&id=swoole_cpu_num) * 1000.
    * Assuming each process occupies `40M` of memory, `100` processes will require `4G` of memory. For how to correctly view the memory usage of processes, please refer to the [official Swoole video tutorial](https://course.swoole-cloud.com/course-video/85).
### max_request

?> **Set the maximum number of tasks for the `worker` process.** [Default value: `0`, meaning the process will not exit]

?> When a `worker` process has processed more tasks than this value, it will automatically exit. Exiting the process will release all memory and resources.

!> The main purpose of this parameter is to solve PHP process memory leak issues caused by improper programming. PHP applications may have slow memory leaks that cannot be identified or fixed. Setting `max_request` temporarily may help with this issue. However, it is essential to locate the memory leak in the code and fix it instead of relying solely on this approach. You can use [Swoole Tracker](https://course.swoole-cloud.com/course-video/92) to discover the leaking code.

  * **Tips**

    * Reaching `max_request` does not necessarily lead to immediate process closure; refer to [max_wait_time](/server/setting?id=max_wait_time).
    * Under [SWOOLE_BASE](/learn?id=swoole_base) mode, when the `max_request` is reached, restarting the process will disconnect client connections.

  !> When a fatal error occurs within the `worker` process or when `exit` is manually executed, the process will exit automatically. The `master` process will then start a new `worker` process to continue handling requests.
### max_conn / max_connection

?> **サーバープログラムで許可される最大接続数。**【デフォルト値：`ulimit -n`】

?> 例: `max_connection => 10000`, このパラメータは`Server`で維持できる最大の`TCP`接続数を設定するために使用されます。この数を超えると、新しい接続が拒否されます。

  * **ヒント**

    * **デフォルト設定**

      * アプリケーションレベルで`max_connection`が設定されていない場合、デフォルト設定として`ulimit -n`の値が使用されます。
      * `4.2.9`以上のバージョンでは、`ulimit -n`が`100000`を超えると、一部のシステムが`ulimit -n`を`100万`に設定しているため、多くのメモリが割り当てられ、起動に失敗するため、デフォルト設定が`100000`になります。

    * **最大限度**

      * `max_connection`を`1M`を超えないようにしてください。

    * **最小設定**
    
      * このオプションを小さな値に設定すると、下位レベルでエラーが発生し、`ulimit -n`の値に設定されます。
      * 最小値は`(worker_num + task_worker_num) * 2 + 32`です。

    ```shell
    serv->max_connection is too small.
    ```

    * **メモリ使用量**

      * `max_connection`パラメータを調整しすぎないようにし、マシンのメモリ状況に基づいて設定してください。`Swoole`はこの数値に基づいて一度に大きなメモリを割り当て、`Connection`情報を保存します。1つの`TCP`接続の`Connection`情報は`224`バイトを占有します。

  * **注意**

  !> `max_connection`は操作システムの`ulimit -n`の値を超えてはならず、そうでない場合は警告メッセージが表示され、`ulimit -n`の値にリセットされます。

  ```shell
  WARN swServer_start_check: serv->max_conn is exceed the maximum value[100000].

  WARNING set_max_connection: max_connection is exceed the maximum value, it's reset to 10240
  ```
### task_worker_num

?> **Configure the number of [Task workers](/learn?id=taskworker-process).**

?> Enabling this parameter will enable the `task` feature. Therefore, the `Server` must register the [onTask](/server/events?id=ontask) and [onFinish](/server/events?id=onfinish) 2 event callback functions. Without registration, the server program will not start.

  * **Tip**

    * [Task workers](/learn?id=taskworker-process) are synchronous and blocking.

    * The maximum value should not exceed [swoole_cpu_num()](/functions?id=swoole_cpu_num) * 1000
    
    * **Calculation method**
      * The processing time of a single `task`, such as `100ms`, then one process can handle `1/0.1=10` tasks in one second
      * The speed of `task` delivery, such as generating `2000` tasks per second
      * `2000/10=200`, set `task_worker_num => 200` is needed to enable `200` Task workers

  * **Caution**

    !> - The `Swoole\Server->task` method cannot be used within [Task workers](/learn?id=taskworker-process)
### task_ipc_mode

?> **[Task Worker](/learn?id=taskworker-process)と`Worker`プロセス間の通信方法を設定します。** 【デフォルト値: `1`】

?> [SwooleにおけるIPC通信](/learn?id=what-is-ipc)を先にお読みください。

モード | 役割
---|---
1 | `Unix Socket`通信を使用【デフォルトモード】
2 | `sysvmsg`メッセージキュー通信を使用
3 | `sysvmsg`メッセージキュー通信を使用し、競合モードを設定

  * **ヒント**

    * **モード `1`**
      * モード `1`を使用すると、指定された`dst_worker_id`を使用して、[task](/server/methods?id=task)および[taskwait](/server/methods?id=taskwait)メソッドでタスクを指定した`Task Worker`に配信できます。
      * `dst_worker_id`が`-1`に設定されている場合、バックエンドは各 [Task Worker](/learn?id=taskworker-process)の状態を確認し、空いているプロセスにタスクを配信します。

    * **モード `2`、`3`**
      * メッセージキューモードは、オペレーティングシステムが提供するメモリキューを使用してデータを保存します。`mssage_queue_key`が指定されていない場合、プライベートキューが使用され、`Server`プログラムが終了するとメッセージキューが削除されます。
      * メッセージキュー`Key`が指定されている場合、`Server`プログラムが終了してもメッセージキュー内のデータは削除されません。そのため、プロセスを再起動してもデータを取得できます。
      * `ipcrm -q`コマンドを使用してメッセージキュー`ID`を手動で削除できます。
      * `モード2`と`モード3`の違いは、`モード2`が配信をサポートする点です。`$serv->task($data, $task_worker_id)`を使用して、どの [taskプロセス](/learn?id=taskworker-process)に送信するかを指定できます。`モード3`は完全な競合モードであり、[taskプロセス](/learn?id=taskworker-process)がキューを競争し、配信先の`ID`を指定できないため、`task/taskwait`で`$task_worker_id`を指定しても、`モード3`では無効です。

  * **注意**

    !> - `モード3`は[sendMessage](/server/methods?id=sendMessage)メソッドに影響を与え、[sendMessage](/server/methods?id=sendMessage)で送信されるメッセージはランダムに1つの[taskプロセス](/learn?id=taskworker-process)に取得されます。
    - メッセージキュー通信を使用すると、`Task Worker`プロセスの処理能力が投稿速度よりも低い場合、`Worker`プロセスがブロックされる可能性があります。
    - メッセージキュー通信を使用すると、taskプロセスは協力のサポートを受けられず( [task_enable_coroutine](/server/setting?id=task_enable_coroutine)を有効にすること)。
### task_max_request

?> **Setting the maximum number of tasks for [task processes](/learn?id=task-worker-process).** 【Default: `0`】

taskプロセスの最大タスク数を設定します。この値を超えるタスクを処理した後、taskプロセスは自動的に終了します。このパラメータはPHPプロセスのメモリオーバーフローを防ぐためのものです。プロセスが自動的に終了しないようにしたい場合は、0に設定できます。
### task_tmpdir

?> **Set the temporary directory for task data.** [Default value: Linux `/tmp` directory]

?> In `Server`, if the submitted data exceeds `8180` bytes, a temporary file will be used to store the data. The `task_tmpdir` is used to specify the location for storing temporary files.

  * **Tip**

    * By default, the underlying system uses the `/tmp` directory to store `task` data. If your `Linux` kernel version is too low and the `/tmp` directory is not a memory file system, you can set it to `/dev/shm/`.
    * If the `task_tmpdir` directory does not exist, the underlying system will automatically attempt to create it.

  * **Note**

    !> - If creation fails, `Server->start` will also fail.
### task_enable_coroutine

?> **`Task` coroutine support enabled.**【Default: `false`】, supported since v4.2.12

?> When enabled, coroutines and a [coroutine scheduler](/coroutine/scheduler) will be automatically created in the [onTask](/server/events?id=ontask) callback, allowing PHP code to directly use coroutine APIs.

  * **Example**

```php
$server->on('Task', function ($serv, Swoole\Server\Task $task) {
    // Which Worker process the task comes from
    $task->worker_id;
    // Task ID
    $task->id;
    // Task type, taskwait, task, taskCo, taskWaitMulti may use different flags
    $task->flags;
    // Task data
    $task->data;
    // Delivery time, added in v4.6.0
    $task->dispatch_time;
    // Coroutine API
    co::sleep(0.2);
    // Complete the task, finish and return data
    $task->finish([123, 'hello']);
});
```

  * **Note**

    !> - `task_enable_coroutine` can only be used when [enable_coroutine](/server/setting?id=enable_coroutine) is `true`  
    - When `task_enable_coroutine` is enabled, `Task` worker processes support coroutines  
    - When `task_enable_coroutine` is not enabled, only synchronous blocking is supported
### task_use_object/task_object :id=task_use_object

?> **Task callback format using object-oriented style.**【Default: `false`】

?> When set to `true`, the [onTask](/server/events?id=ontask) callback will change to object mode.

  * **Example**

```php
<?php

$server = new Swoole\Server('127.0.0.1', 9501);
$server->set([
    'worker_num'      => 1,
    'task_worker_num' => 3,
    'task_use_object' => true,
//    'task_object' => true, // Alias added in version 4.6.0
]);
$server->on('receive', function (Swoole\Server $server, $fd, $tid, $data) {
    $server->task(['fd' => $fd,]);
});
$server->on('Task', function (Swoole\Server $server, Swoole\Server\Task $task) {
    // Here, $task is of the Swoole\Server\Task object type
    $server->send($task->data['fd'], json_encode($server->stats()));
});
$server->start();
```
### dispatch_mode

?> **データパケットの配信モード。**【デフォルト値: `2`】

モード値 | モード | 機能
---|---|---
1 | ラウンドロビンモード | 受信されたパケットは各`Worker`プロセスにラウンドロビンで割り当てられます。
2 | 固定モード | 接続のファイル記述子に基づいて`Worker`を割り当てます。これにより、同じ接続からのデータは同じ`Worker`だけが処理することが保証されます。
3 | プリエンプティブモード | マスタープロセスは`Worker`のアイドル状態に基づいてディスパッチを選択し、アイドル状態の`Worker`にだけ配信されます。
4 | IP割り当て | クライアントの`IP`に基づいてモジュロ`hash`を取り、特定の`Worker`プロセスに割り当てます。<br>同じソースIPからの接続データが常に同じ`Worker`プロセスに割り当てられることを保証します。アルゴリズムは `inet_addr_mod(クライアントIP, worker_num)`
5 | UID割り当て | ユーザーコードで[Server->bind()](/server/methods?id=bind)を呼び出して接続に`1`つの`uid`をバインドする必要があります。その後、下位層は`UID`の値に基づいて異なる`Worker`プロセスに割り当てます。<br>アルゴリズムは `UID % worker_num` で、文字列を`UID`として使用する場合は `crc32(UID_STRING)` を使用できます。
7 | streamモード | アイドルの`Worker`は接続を`accept`し、[Reactor](/learn?id=reactor糸)の新しいリクエストを受け入れます。

  * **ヒント**

    * **ご利用方法**
    
      * 状態を持たない`Server`は`1`または`3`を使用できます。同期ブロック`Server`は`3`を使用し、非同期非ブロッキング`Server`は`1`を使用します。
      * 状態を持つ場合は`2`、`4`、`5`を使用します。
      
    * **UDPプロトコル**
    
      * `dispatch_mode=2/4/5`の場合は固定割り当てであり、下位層はクライアントの`IP`をハッシュして異なる`Worker`プロセスに割り当てます
      * `dispatch_mode=1/3`の場合は異なる`Worker`プロセスにランダムに割り当てられます
      * `inet_addr_mod`関数

```
    function inet_addr_mod($ip, $worker_num) {
        $ip_parts = explode('.', $ip);
        if (count($ip_parts) != 4) {
            return false;
        }
        $ip_parts = array_reverse($ip_parts);
    
        $ip_long = 0;
        foreach ($ip_parts as $part) {
            $ip_long <<= 8;
            $ip_long |= (int) $part;
        }
    
        return $ip_long % $worker_num;
    }
```

    * **BASEモード**
    
      * [SWOOLE_BASE](/learn?id=swoole_base) モードでの`dispatch_mode`設定は無効です。なぜなら`BASE`にはタスクデリゲーションがないためで、クライアントからのデータを受信するとすぐに現在のスレッド/プロセスで[onReceive](/server/events?id=onreceive)をコールバックします。

  * **注意**

    !> -`dispatch_mode=1/3`の場合、下位層では`onConnect/onClose`イベントがブロックされます。これらの2つのモードでは`onConnect/onClose/onReceive`の順序が保証されません。  
    -リクエストレスポンス形式でないサーバープログラムでは、モード`1`または`3`を使用しないでください。たとえば、httpサービスはレスポンス形式であるため、`1`または`3`を使用できます。TCP長接続状態の場合は`1`または`3`を使用しないでください。
### dispatch_func

?> Setting the `dispatch` function, `Swoole` internally builds in `6` types of [dispatch_mode](/server/setting?id=dispatch_mode). If it still doesn't meet your requirements, you can write a `C++` function or `PHP` function to implement the `dispatch` logic.

  * **Usage**

```php
$server->set(array(
  'dispatch_func' => 'my_dispatch_function',
));
```

  * **Tips**

    * After setting `dispatch_func`, the underlying layer will automatically ignore the `dispatch_mode` configuration
    * If the function corresponding to `dispatch_func` does not exist, the underlying layer will throw a fatal error
    * If you need to `dispatch` a package larger than `8K`, `dispatch_func` can only get contents of `0-8180` bytes

  * **Write PHP function**

    ?> Due to the inability of `ZendVM` to support multi-threaded environments, even if multiple [Reactor](/learn?id=reactor-line) threads are set, only one `dispatch_func` can be executed at a time. Therefore, when executing this PHP function at the underlying layer, a locking operation will be performed, which may lead to contention for locks. Do not perform any blocking operations in the `dispatch_func`, as it will cause the `Reactor` thread group to stop working.

    ```php
    $server->set(array(
        'dispatch_func' => function ($server, $fd, $type, $data) {
            var_dump($fd, $type, $data);
            return intval($data[0]);
        },
    ));
    ```

    * `$fd` is the unique identifier of the client connection, and you can use `Server::getClientInfo` to get connection information
    * `$type` type of data, `0` indicates data sent from the client, `4` indicates the client connection being established, `3` indicates the client connection being closed
    * `$data` data content, note: if parameters such as `HTTP`, `EOF`, `Length`, etc., are enabled for protocol processing, the underlying layer will splice the packets. However, only the first 8K content of the packet can be passed to the `dispatch_func` function, and the complete packet content cannot be obtained.
    * It is **mandatory** to return a number from `0` to `(server->worker_num - 1)`, indicating the target worker process `ID` for packet delivery
    * If less than `0` or greater than or equal to `server->worker_num`, it is an abnormal target `ID`, and the dispatched data will be discarded

  * **Write C++ function**

    **In other PHP extensions, use swoole_add_function to register a length function with the Swoole engine.**

    ?> The underlying layer does not lock when calling C++ functions, and the caller needs to ensure thread safety by themselves.

    ```c++
    int dispatch_function(swServer *serv, swConnection *conn, swEventData *data);

    int dispatch_function(swServer *serv, swConnection *conn, swEventData *data)
    {
        printf("cpp, type=%d, size=%d\n", data->info.type, data->info.len);
        return data->info.len % serv->worker_num;
    }

    int register_dispatch_function(swModule *module)
    {
        swoole_add_function("my_dispatch_function", (void *) dispatch_function);
    }
    ```

    * The `dispatch` function must return the target `worker` process `id`
    * The returned `worker_id` must not exceed `server->worker_num`, otherwise a segmentation fault will occur at the underlying layer
    * Returning a negative number `(return -1)` indicates discarding this data packet
    * `data` can read the event type and length
    * `conn` is the connection information. If it is a `UDP` packet, `conn` is `NULL`

  * **Note**

    !> - `dispatch_func` is only effective in [SWOOLE_PROCESS](/learn?id=swoole_process) mode, available for servers of type [UDP/TCP/UnixSocket](/server/methods?id=__construct)  
    - The returned `worker_id` must not exceed `server->worker_num`, otherwise a segmentation fault will occur
### message_queue_key

?> **Set the key for the message queue.** 【Default value: `ftok($php_script_file, 1)`】

?> This is used only when [task_ipc_mode](/server/setting?id=task_ipc_mode) is set to 2/3. The key set is only used as the key for the `Task` task queue, refer to [IPC communication under Swoole](/learn?id=ipc). 

?> The `task` queue will not be destroyed after the `server` is terminated. When the program restarts, the [task processes](/learn?id=taskworker-process) will continue to handle tasks in the queue. If you do not want the old `Task` tasks to be executed after the program restarts, you can manually delete this message queue. 

```shell
ipcs -q 
ipcrm -Q [msgkey]
```
### daemonize

?> **デーモン化**【デフォルト値：`false`】

?> `daemonize => true` に設定すると、プログラムはデーモンとしてバックグラウンドで実行されます。 長時間実行されるサーバーサイドプログラムは、この設定を有効にする必要があります。  
デーモンを有効にしない場合、sshターミナルを終了するとプログラムが停止します。

  * **ヒント**

    * デーモンを有効にすると、標準入力と出力は `log_file` にリダイレクトされます。
    * `log_file` が設定されていない場合、`/dev/null` にリダイレクトされ、画面に表示された情報はすべて破棄されます。
    * デーモンを有効にすると、`CWD`（カレントディレクトリ）環境変数の値が変更され、相対パスのファイル読み書きがエラーになります。`PHP` プログラムでは絶対パスを使用する必要があります。

    * **systemd**

      * `Swoole` サービスを管理する際に `systemd` または `supervisord` を使用する場合、`daemonize => true` を設定しないでください。主な理由は `systemd` のメカニズムが `init` と異なるからです。`init` プロセスの `PID` は `1` であり、プログラムが `daemonize` を使用すると、端末から切り離され、最終的に `init` プロセスによって管理されるようになり、`init` との関係が親子プロセスの関係になります。
      * しかし、`systemd` は別のバックグラウンドプロセスを起動し、自前で他のサービスプロセスを `fork` して管理するため、`daemonize` は必要ありません。むしろ `daemonize => true` を使用すると、`Swoole` プログラムがその管理プロセスとの親子プロセス関係を失います。
### バックログ

?> **`Listen`キューの長さを設定します**

?> たとえば`backlog => 128`のように、このパラメータは同時に`accept`を待機する接続の最大数を決定します。

  * **`TCP`の`backlog`について**

    ?> `TCP`には3ウェイハンドシェイクのプロセスがあります。クライアント `syn=>サーバ` `syn+ack=>クライアント` `ack`。サーバがクライアントから`ack`を受け取ると、接続を`accept queue`というキューに置くことになります（注1）。  
    キューのサイズは`backlog`パラメータと設定された`somaxconn`の最小値によって決定されます。`ss -lt`コマンドを使用して最終的な`accept queue`のキューサイズを確認できます。`Swoole`のメインプロセスは`accept`を呼び出し（注2）、  
    `accept queue`から取り出します。 `accept queue`がいっぱいになると、接続が成功する可能性があります（注4）、  
    失敗することもあります。失敗した場合、クライアント側の挙動は接続がリセットされる（注3）  
    または接続がタイムアウトすることになりますが、サーバーは失敗したレコードを記録し、 `netstat -s|grep 'times the listen queue of a socket overflowed` を使用してログを確認できます。上記の現象が発生した場合は、この値を増やす必要があります。 `Swoole`の幸運なことにはSWOOLE_PROCESSモードは`PHP-FPM/Apache`などとは異なり、接続の待ち行列を解決するために`backlog`に依存しません。したがって、基本的には上記の現象に遭遇することはほとんどありません。

    * 注1: `linux2.2`以降、ハンドシェイクプロセスは`syn queue`と`accept queue`の2つのキューに分かれ、`syn queue`の長さは`tcp_max_syn_backlog`によって決定されます。
    * 注2: 高いバージョンのカーネルでは、`accept4`が呼び出されます。これにより、1回の`set no block`システムコールが節約されます。
    * 注3: クライアントは`syn+ack`パケットを受信すると接続が成功したとみなしますが、実際にはサーバーは半接続状態にあり、クライアントに`rst`パケットを送信する可能性があります。クライアントの挙動は`Connection reset by peer`になります。
    * 注4: 成功はTCPの再送制御によって行われ、関連する設定は`tcp_synack_retries`および`tcp_abort_on_overflow`です。低レベルのTCPメカニズムを詳しく学習したい場合は、[Swoole公式ビデオチュートリアル](https://course.swoole-cloud.com/course-video/3)を参照してください。
### open_tcp_keepalive

`TCP`にはデッドコネクションを検出する`Keep-Alive`メカニズムがあります。アプリケーションレイヤがデッドリンクの周期に敏感でない場合や、ハートビートメカニズムを実装していない場合は、オペレーティングシステムが提供する`keepalive`メカニズムを使用してデッドリンクを切断できます。
[Server->set()](/server/methods?id=set)で`open_tcp_keepalive => true`を設定すると、`TCP keepalive`が有効になります。
また、`keepalive`のディテールを調整するために`3`つのオプションがあります。詳細は[Swoole公式ビデオコース](https://course.swoole-cloud.com/course-video/10)を参照してください。

  * **オプション**

     * **tcp_keepidle**

        単位：秒。接続が`n`秒間データリクエストを受信しない場合、この接続は探索されます。

     * **tcp_keepcount**

        探索回数。この回数を超えると接続が`close`されます。

     * **tcp_keepinterval**

        探索間隔、単位：秒。

  * **例**

```php
$serv = new Swoole\Server("192.168.2.194", 6666, SWOOLE_PROCESS);
$serv->set(array(
    'worker_num' => 1,
    'open_tcp_keepalive' => true,
    'tcp_keepidle' => 4, // データ転送が4秒間ない場合にチェックを開始
    'tcp_keepinterval' => 1, // 1秒ごとに探索
    'tcp_keepcount' => 5, // 探索回数を超えるとまだレスポンスがない場合に接続を閉じる
));

$serv->on('connect', function ($serv, $fd) {
    var_dump("Client:Connect $fd");
});

$serv->on('receive', function ($serv, $fd, $reactor_id, $data) {
    var_dump($data);
});

$serv->on('close', function ($serv, $fd) {
  var_dump("close fd $fd");
});

$serv->start();
```
### heartbeat_check_interval

?> **ハートビートチェックを有効にする**【デフォルト値：`false`】

?> このオプションは、指定した間隔で繰り返しチェックすることを示します。単位は秒です。 例えば `heartbeat_check_interval => 60` は、全ての接続を`60`秒ごとに処理し、その接続が過去`120`秒（`heartbeat_idle_time`が設定されていない場合、デフォルトで`interval`の2倍）以内にサーバーにデータを送信していない場合、その接続を強制的に切断します。 設定されていない場合、ハートビートは有効にならず、この設定はデフォルトで閉じています。[Swoole公式ビデオチュートリアル](https://course.swoole-cloud.com/course-video/10) を参照してください。

  * **ヒント**
    * `Server`はクライアントに対してハートビートパケットを自動的に送信しません。代わりにクライアントがハートビートを送信するのを待ちます。サーバー側の`heartbeat_check`は単に接続が前回データを送信した時刻をチェックし、制限を超えた場合に接続を切断します。
    * ハートビートチェックによって切断された接続は、依然として[onClose](/server/events?id=onclose) イベントコールバックをトリガーします。

  * **注意**

    !> `heartbeat_check` は`TCP`接続のみをサポートします。
### heartbeat_idle_time

?> **接続が許可される最大のアイドル時間**

?> `heartbeat_check_interval`と一緒に使用する必要があります

```php
array(
    'heartbeat_idle_time'      => 600, // 表示一个连接如果600秒内未向服务器发送任何数据，此连接将被强制关闭
    'heartbeat_check_interval' => 60,  // 表示每60秒遍历一次
);
```

  * **ヒント**

    * `heartbeat_idle_time`を有効にすると、サーバーはクライアントに対してデータパケットをアクティブに送信しません
    * `heartbeat_idle_time`のみを設定して`heartbeat_check_interval`を設定しない場合、内部でハートビート検査スレッドは作成されません。`PHP`コードで`heartbeat`メソッドを呼び出してタイムアウトした接続を手動で処理できます
### open_eof_check

?> **`EOF` detection enabled**【default: `false`】，ref. [TCP Data Pack Boundary Issue](/learn?id=tcp-data-pack-boundary-issue)

?> This option checks the data sent by the client connection. The data packet will only be delivered to the `Worker` process when the end of the packet matches the specified string. Otherwise, the data packets will be concatenated until they exceed the buffer or timeout occurs. In case of errors, the underlying system will consider it a malicious connection, discard the data, and forcibly close the connection.  
Common protocols like `Memcache/SMTP/POP` end with `\r\n`, and this configuration can be used with them. Enabling this option ensures that the `Worker` process always receives one or more complete data packets at a time.

```php
array(
    'open_eof_check' => true,   //enable EOF check
    'package_eof'    => "\r\n", //set EOF
)
```

  * **Note**

    !> This setting only applies to `STREAM` type `Socket`, such as [TCP, Unix Socket Stream](/server/methods?id=__construct)   
    The `EOF` detection does not search for the `EOF` string in the middle of the data. Therefore, the `Worker` process may receive multiple data packets at the same time, and you need to manually `explode("\r\n", $data)` in the application code to split the data packets.
### open_eof_split

?> **Enable `EOF` automatic packet splitting**

?> When `open_eof_check` is set, multiple data may be combined in one packet. Parameter `open_eof_split` can solve this problem, refer to [TCP data packet boundary problem](/learn?id=tcp数据包边界问题).

?> Setting this parameter requires traversing the entire content of the packet to find `EOF`, so it will consume a lot of `CPU` resources. Assuming each data packet is `2M`, with `10000` requests per second, this may result in `20G` CPU character matching instructions.

```php
array(
    'open_eof_split' => true,   //Enable EOF_SPLIT check
    'package_eof'    => "\r\n", //Set EOF
)
```

  * **Tips**

    * When `open_eof_split` parameter is enabled, the underlying will search for `EOF` in the middle of the data packet and split the data packet. [onReceive](/server/events?id=onreceive) will only receive a data packet ending with the `EOF` string each time.
    
    * After enabling the `open_eof_split` parameter, it will take effect regardless of whether or not the `open_eof_check` parameter is set.

    * **Differences from `open_eof_check`**
    
        * `open_eof_check` only checks if the end of the received data is `EOF`, so it has the best performance with almost no consumption.
        * `open_eof_check` cannot solve the problem of multiple data packets being combined, for example, when sending two data with `EOF` at the same time, the underlying may return all at once.
        * `open_eof_split` will compare the data byte by byte from left to right to find `EOF` in the data for packet splitting, which has poor performance. However, only one data packet will be returned each time.
### package_eof

?> **Set the `EOF` string.** Refer to [TCP data packet boundary issue](/learn?id=tcp数据包边界问题)

?> Need to be used in conjunction with `open_eof_check` or `open_eof_split`.

  * **Note**

    !> `package_eof` allows a maximum of only `8` bytes of string.
### open_length_check

?> **Open package length check feature** [Default value: `false`], refer to [TCP package boundary issue](/learn?id=tcp数据包边界问题)

?> Length checking provides parsing for protocols with a fixed package head + body format. When enabled, it ensures that the `Worker` process will always receive a complete data packet in the [onReceive](/server/events?id=onreceive) event.  
Length checking protocol requires calculating the length only once, with data processing involving only pointer offsets, providing very high performance. **Recommended for use**.

  * **Tip**

    * **Length protocol provides 3 options to control protocol details.**

      ?> This configuration is only effective for `STREAM` type `Socket`, such as [TCP, Unix Socket Stream](/server/methods?id=__construct)

      * **package_length_type**

        ?> A field in the package header serves as the value for the package length, supporting 10 types of lengths at the underlying level. Please refer to [package_length_type](/server/setting?id=package_length_type)

      * **package_body_offset**

        ?> Calculates the length starting from the nth byte, generally in 2 situations:

        * The value of `length` includes the entire package (header + body), with `package_body_offset` set to `0`.
        * The header length is `N` bytes, the value of `length` does not include the header but only the body, with `package_body_offset` set to `N`.

      * **package_length_offset**

        ?> The position of the `length` value within the package header.

        * Example:

        ```c
        struct
        {
            uint32_t type;
            uint32_t uid;
            uint32_t length;
            uint32_t serid;
            char body[0];
        }
        ```
        
    ?> In the above communication protocol design, the header length is `4` integers, `16` bytes, and the `length` value is at the position of the 3rd integer. Therefore, `package_length_offset` is set to `8`, bytes `0-3` are for `type`, bytes `4-7` are for `uid`, bytes `8-11` are for `length`, and bytes `12-15` are for `serid`.

    ```php
    $server->set(array(
      'open_length_check'     => true,
      'package_max_length'    => 81920,
      'package_length_type'   => 'N',
      'package_length_offset' => 8,
      'package_body_offset'   => 16,
    ));
    ```
### package_length_type

?> **長さの値のタイプ**について、1文字の引数を受け取り、`PHP`の [pack](http://php.net/manual/zh/function.pack.php) 関数と同じです。

現在、`Swoole` は10種類のタイプをサポートしています：

| 文字引数 | 役割           |
| ---     | ---            |
| c       | 有符号、1バイト |
| C       | 無符号、1バイト |
| s       | 有符号、ホストバイトオーダー、2バイト |
| S       | 無符号、ホストバイトオーダー、2バイト |
| n       | 無符号、ネットワークバイトオーダー、2バイト |
| N       | 無符号、ネットワークバイトオーダー、4バイト |
| l       | 有符号、ホストバイトオーダー、4バイト（小文字のL） |
| L       | 無符号、ホストバイトオーダー、4バイト（大文字のL） |
| v       | 無符号、リトルエンディアンバイトオーダー、2バイト |
| V       | 無符号、リトルエンディアンバイトオーダー、4バイト |
### package_length_func

?> **長さ解析関数を設定します**

?> `C++`または`PHP`の`2`つの種類の関数をサポートします。長さ関数は整数値を返さなければなりません。

戻り値 | 意味
---|---
0を返す | 長さデータが足りないため、さらにデータを受信する必要があります
-1を返す | データエラーが発生し、接続が自動的に閉じられます
パケットの長さ値を返す（ヘッダーとボディー全体の長さを含む） | パケットを自動的に組み立てて、コールバック関数に返します

  * **ヒント**

    * **使用方法**

    ?> 実装原理は、最初に一部のデータを読み取り、そのデータに長さ値が含まれていることです。そして、この長さを返して、残りのデータの受信とパケットのディスパッチを底層が完了します。

    * **PHPの長さ解析関数**

    ?> `ZendVM`はマルチスレッド環境での実行をサポートしないため、底層では`PHP`の長さ関数を並行実行するのを避けるために自動的に`Mutex`ミューテックスを使用します。`1.9.3`以降で使用可能。

    !> 長さ解析関数でブロッキング`IO`操作を実行しないでください。すべての[Reactor](/learn?id=reactor線)スレッドがブロックされる可能性があります

    ```php
    $server = new Swoole\Server("127.0.0.1", 9501);
    
    $server->set(array(
        'open_length_check'   => true,
        'dispatch_mode'       => 1,
        'package_length_func' => function ($data) {
          if (strlen($data) < 8) {
              return 0;
          }
          $length = intval(trim(substr($data, 0, 8)));
          if ($length <= 0) {
              return -1;
          }
          return $length + 8;
        },
        'package_max_length'  => 2000000,  //プロトコルの最大長さ
    ));
    
    $server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        var_dump($data);
        echo "#{$server->worker_id}>> received length=" . strlen($data) . "\n";
    });
    
    $server->start();
    ```

    * **C++の長さ解析関数**

    ?> 他のPHP拡張では、`swoole_add_function`を使用して長さ関数を`Swoole`エンジンに登録します。
   
    !> C++の長さ関数は呼び出し時に底層がロックされないため、呼び出し側がスレッドセーフを保証する必要があります

    ```c++
    #include <string>
    #include <iostream>
    #include "swoole.h"
    
    using namespace std;
    
    int test_get_length(swProtocol *protocol, swConnection *conn, char *data, uint32_t length);
    
    void register_length_function(void)
    {
        swoole_add_function((char *) "test_get_length", (void *) test_get_length);
        return SW_OK;
    }
    
    int test_get_length(swProtocol *protocol, swConnection *conn, char *data, uint32_t length)
    {
        printf("cpp, size=%d\n", length);
        return 100;
    }
    ```  
### package_max_length

?> **Set the maximum size of the data package in bytes.** [Default value: `2M` which is `2 * 1024 * 1024`, minimum value is `64K`]

?> After enabling [open_length_check](/server/setting?id=open_length_check)/[open_eof_check](/server/setting?id=open_eof_check)/[open_eof_split](/server/setting?id=open_eof_split)/[open_http_protocol](/server/setting?id=open_http_protocol)/[open_http2_protocol](/http_server?id=open_http2_protocol)/[open_websocket_protocol](/server/setting?id=open_websocket_protocol)/[open_mqtt_protocol](/server/setting?id=open_mqtt_protocol) and other protocol parsing, the `Swoole` underlying layer will concatenate data packets. At this time, when the data packet is not received completely, all data is saved in memory.
Therefore, you need to set `package_max_length`, which is the maximum memory size a data packet is allowed to occupy. If there are 10,000 `TCP` connections sending data at the same time, each data packet is `2M`, then in the worst case scenario, it will occupy `20G` of memory space.

  * **Hint**

    * `open_length_check`: When a packet length exceeds `package_max_length`, this data will be directly discarded, and the connection will be closed without occupying any memory.
    * `open_eof_check`: Because it is impossible to know the data packet length in advance, the received data will continue to be stored in memory, increasing continuously. When the memory usage exceeds `package_max_length`, this data will be directly discarded, and the connection will be closed.
    * `open_http_protocol`: The maximum allowed length for a `GET` request is `8K`, and this configuration cannot be modified. For `POST` requests, the `Content-Length` will be checked, and if `Content-Length` exceeds `package_max_length`, this data will be directly discarded, an `http 400` error will be sent, and the connection will be closed.

  * **Note**

    !> This parameter should not be set too large, as it may occupy a significant amount of memory.
### open_http_protocol

?> **Enable the `HTTP` protocol processing.**【Default value: `false`】

?> Enabling `HTTP` protocol processing, [Swoole\Http\Server](/http_server) will automatically enable this option. Setting it to `false` means turning off `HTTP` protocol processing.
### open_mqtt_protocol

?> **Enable handling of `MQTT` protocol.**【Default: `false`】

?> When enabled, it will parse the `MQTT` packet header, and the `worker` process will return a complete `MQTT` data packet each time during [onReceive](/server/events?id=onreceive).

```php
$server->set(array(
  'open_mqtt_protocol' => true
));
```
### open_redis_protocol

?> **Enable `Redis` protocol processing.**【Default value: `false`】

?> When enabled, it will parse the `Redis` protocol, and the `worker` process will return a complete `Redis` data packet each time [onReceive](/server/events?id=onreceive) is triggered. It is recommended to use [Redis\Server](/redis_server)

```php
$server->set(array(
  'open_redis_protocol' => true
));
```
### open_websocket_protocol

?> **Enable the `WebSocket` protocol processing.** [Default: `false`]

?> Enable the processing of the `WebSocket` protocol; the [Swoole\WebSocket\Server](websocket_server) will automatically enable this option. Setting it to `false` means closing `websocket` protocol processing.  
When you set the `open_websocket_protocol` option to `true`, the `open_http_protocol` will also be automatically set to `true`.
### open_websocket_close_frame

?> **Websocketプロトコルのクローズフレームを有効にします。**【デフォルト値：`false`】

?> （`opcode`が`0x08`のフレーム）を`onMessage`コールバックで受信します。

?> 有効にすると、`WebSocketServer`の`onMessage`コールバックでクライアントまたはサーバーから送信されたクローズフレームを受信でき、開発者はそれを自身で処理できます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->set(array("open_websocket_close_frame" => true));

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x08) {
        echo "Close frame received: Code {$frame->code} Reason {$frame->reason}\n";
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {});

$server->start();
```
### open_tcp_nodelay

?> **`open_tcp_nodelay`を有効にします。**【デフォルト値：`false`】

?> 有効にすると、データを送信する際に`TCP`接続で`Nagle`統合アルゴリズムが無効になり、すぐに対向のTCP接続に送信されます。一部のシナリオでは、コマンドライン端末の場合など、コマンドをすぐにサーバーに送信する必要がある場合があり、応答速度が向上する可能性があります。Nagleアルゴリズムについては、自分で調べてみてください。
### open_cpu_affinity

？> **CPUアフィニティ設定を有効にします。** 【デフォルト`false`】

？> マルチコアハードウェアプラットフォームでは、この機能を有効にすると、`Swoole`の`reactorスレッド`/`workerプロセス`を固定の1つのコアにバインドします。プロセス/スレッドが複数のコア間で切り替わることを避け、`CPU`の`Cache`のヒット率を向上させることができます。

  * **ヒント**

    * **tasksetコマンドを使用してプロセスのCPUアフィニティ設定を確認します：**

    ```bash
    taskset -p プロセスID
    pid 24666's current affinity mask: f
    pid 24901's current affinity mask: 8
    ```

    > マスクはマスク番号で、`bit`ごとに1つの`CPU`コアが対応しています。特定のビットが`0`の場合、そのコアにバインドされ、プロセスはその`CPU`にスケジュールされます。 `0`の場合、プロセスはその`CPU`にスケジュールされません。 例では、`pid`が`24666`のプロセスの`mask = f`は`CPU`にバインドされていないことを示し、オペレーティングシステムはこのプロセスをどの`CPU`コアにもスケジュールします。 `pid`が`24901`のプロセスの`mask = 8`では、`8`を2進数に変換すると `1000`になり、このプロセスは第`4`の`CPU`コアにバインドされています。
### cpu_affinity_ignore

?> **IO-intensive programs handle all network interrupts using CPU0. If the network IO is heavy and CPU0 is overloaded, the network interrupts may not be processed in time, leading to a decrease in network packet processing capability.**

?> If this option is not set, Swoole will use all CPU cores, and the underlying system will set CPU binding based on the `reactor_id` or `worker_id` and the number of CPU cores.  
If the kernel and network card support multiple queues, network interrupts will be distributed to multiple cores, helping to reduce the pressure on network interrupts.

```php
array('cpu_affinity_ignore' => array(0, 1)) // Accepts an array as a parameter, where `array(0, 1)` indicates not using CPU0 and CPU1 to specifically free them up to handle network interrupts.
```

  * **Tips**

    * **View network interrupts**

```shell
[~]$ cat /proc/interrupts 
           CPU0       CPU1       CPU2       CPU3       
  0: 1383283707          0          0          0    IO-APIC-edge  timer
  1:          3          0          0          0    IO-APIC-edge  i8042
  3:         11          0          0          0    IO-APIC-edge  serial
  8:          1          0          0          0    IO-APIC-edge  rtc
  9:          0          0          0          0   IO-APIC-level  acpi
 12:          4          0          0          0    IO-APIC-edge  i8042
 14:         25          0          0          0    IO-APIC-edge  ide0
 82:         85          0          0          0   IO-APIC-level  uhci_hcd:usb5
 90:         96          0          0          0   IO-APIC-level  uhci_hcd:usb6
114:    1067499          0          0          0       PCI-MSI-X  cciss0
130:   96508322          0          0          0         PCI-MSI  eth0
138:     384295          0          0          0         PCI-MSI  eth1
169:          0          0          0          0   IO-APIC-level  ehci_hcd:usb1, uhci_hcd:usb2
177:          0          0          0          0   IO-APIC-level  uhci_hcd:usb3
185:          0          0          0          0   IO-APIC-level  uhci_hcd:usb4
NMI:      11370       6399       6845       6300 
LOC: 1383174675 1383278112 1383174810 1383277705 
ERR:          0
MIS:          0
```

`eth0/eth1` represents the number of network interrupts. If `CPU0 - CPU3` are evenly distributed, it indicates that the network card supports multiple queues. If all interrupts are concentrated on one core, it means that all network interrupts are processed by this CPU. If this CPU exceeds `100%`, the system will be unable to handle network requests. In such cases, it is necessary to set `cpu_affinity_ignore` to free up this CPU specifically for handling network interrupts.

In the example shown above, you should set `cpu_affinity_ignore => array(0)`.

?> You can use the `top` command `->` type `1` to check the usage of each core.

  * **Note**

    !> This option must be set together with `open_cpu_affinity` to take effect.
### tcp_defer_accept

?> **Enable the `tcp_defer_accept` feature** [Default: `false`]

?> It can be set to a number, indicating that `accept` is only triggered when a `TCP` connection has data to send.

```php
$server->set(array(
  'tcp_defer_accept' => 5
));
```

  * **Note**

    * **After enabling the `tcp_defer_accept` feature, the timing of `accept` and [onConnect](/server/events?id=onconnect) will change. If set to `5` seconds:**

      * The `accept` is not triggered immediately after the client connects to the server
      * If the client sends data within `5` seconds, `accept/onConnect/onReceive` will be triggered concurrently
      * If the client does not send any data within `5` seconds, `accept/onConnect` will be triggered
### ssl_cert_file / ssl_key_file :id=ssl_cert_file

?> **SSLトンネル暗号化を設定します。**

?> ファイル名の文字列を指定して、cert証明書とkey秘密鍵のパスを設定します。

  * **ヒント**

    * **`PEM` to `DER` format**

    ```shell
    openssl x509 -in cert.crt -outform der -out cert.der
    ```

    * **`DER` to `PEM` format**

    ```shell
    openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
    ```

  * **注意**

    !> - `HTTPS`アプリケーションでは、ブラウザが証明書を信頼している必要があります。  
    - `wss`アプリケーションでは、`WebSocket`接続を開始するページは `HTTPS` を使用する必要があります。  
    - ブラウザが`SSL`証明書を信頼しない場合、 `wss` を使用できません。  
    - ファイルは`PEM`形式である必要があり、`DER`形式はサポートされていません。`openssl`ツールを使用して変換できます。

    !> `SSL`を使用するには、`Swoole`をコンパイルする際に[--enable-openssl](/environment?id=编译选项)オプションを追加する必要があります。

    ```php
    $server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
    $server->set(array(
        'ssl_cert_file' => __DIR__.'/config/ssl.crt',
        'ssl_key_file' => __DIR__.'/config/ssl.key',
    ));
    ```
### ssl_method

!> このパラメータは[v4.5.4](/version/bc?id=_454)で削除されました。`ssl_protocols`を使用してください。

?> **OpenSSLのトンネル暗号化アルゴリズムを設定します。** [デフォルト値: `SWOOLE_SSLv23_METHOD`]、サポートされるタイプについては[SSL Encryption Methods](/consts?id=ssl-encryption-methods)を参照してください。

?> `Server`と`Client`で使用されるアルゴリズムは同じである必要があります。そうでないと`SSL/TLS`のハンドシェイクが失敗し、接続が切断されます。

```php
$server->set(array(
    'ssl_method' => SWOOLE_SSLv3_CLIENT_METHOD,
));
```
### ssl_protocols

?> **Set the protocol for OpenSSL tunnel encryption.** 【Default value: `0`, supports all protocols】, please refer to [SSL Protocols](/consts?id=ssl-protocols) for supported types.

!> Available in Swoole version >= `v4.5.4`

```php
$server->set(array(
    'ssl_protocols' => 0,
));
```
### ssl_sni_certs

?> **SNI (Server Name Identification) 証明書の設定**

!> Swooleバージョン >= `v4.6.0` で使用可能

```php
$server->set([
    'ssl_cert_file' => __DIR__ . '/server.crt',
    'ssl_key_file' => __DIR__ . '/server.key',
    'ssl_protocols' => SWOOLE_SSL_TLSv1_2 | SWOOLE_SSL_TLSv1_3 | SWOOLE_SSL_TLSv1_1 | SWOOLE_SSL_SSLv2,
    'ssl_sni_certs' => [
        'cs.php.net' => [
            'ssl_cert_file' => __DIR__ . '/sni_server_cs_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_cs_key.pem',
        ],
        'uk.php.net' => [
            'ssl_cert_file' =>  __DIR__ . '/sni_server_uk_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_uk_key.pem',
        ],
        'us.php.net' => [
            'ssl_cert_file' => __DIR__ . '/sni_server_us_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_us_key.pem',
        ],
    ]
]);
```
### ssl_ciphers

?> **Set the OpenSSL encryption algorithm.**【Default Value: `EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH`】

```php
$server->set(array(
    'ssl_ciphers' => 'ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP',
));
```

  * **Hint**

    * When `ssl_ciphers` is set to an empty string, the encryption algorithm will be chosen by `openssl` automatically.
### ssl_verify_peer

?> **サーバーSSL設定において、対向端の証明書を検証するかどうかを設定します。**【デフォルト値：`false`】

?> デフォルトはオフであり、クライアント証明書の検証は行いません。有効にした場合は、`ssl_client_cert_file` オプションも同時に設定する必要があります。
### ssl_allow_self_signed

?> **Allow self-signed certificates.** [Default: `false`]
### ssl_client_cert_file

?> **クライアント証明書を検証するためのルート証明書。**

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set(array(
    'ssl_cert_file'         => __DIR__ . '/config/ssl.crt',
    'ssl_key_file'          => __DIR__ . '/config/ssl.key',
    'ssl_verify_peer'       => true,
    'ssl_allow_self_signed' => true,
    'ssl_client_cert_file'  => __DIR__ . '/config/ca.crt',
));
```

!> `TCP`サービスが検証に失敗すると、下層で自動的に接続が閉じられます。
### ssl_compress

?> **This sets whether `SSL/TLS` compression is enabled.** When using [Co\Client](/coroutine_client/client), it has an alias `ssl_disable_compression`
### ssl_verify_depth

?> **If the certificate chain is too deep, exceeding the value set by this option, the verification will be terminated.**
### ssl_prefer_server_ciphers

?> **サーバー側のクライアントを好む暗号を有効にし、BEAST攻撃を防止します。**
### ssl_dhparam

?> **指定DHE密码器的`Diffie-Hellman`参数。**
### ssl_ecdh_curve

ECDH鍵交換で使用する`curve`を指定します。

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set([
    'ssl_compress'                => true,
    'ssl_verify_depth'            => 10,
    'ssl_prefer_server_ciphers'   => true,
    'ssl_dhparam'                 => '',
    'ssl_ecdh_curve'              => '',
]);
```
### ユーザー

?> **`Worker/TaskWorker`サブプロセスの所有者を設定します。**【デフォルト値：スクリプトを実行するユーザー】

?> サーバーが`1024`以下のポートをリッスンする必要がある場合、`root`権限が必要です。しかし、プログラムが`root`ユーザーで実行されると、コードに脆弱性があると、攻撃者が`root`権限でリモートコマンドを実行できる可能性があり、リスクが非常に高くなります。`user`パラメータが設定されている場合、メインプロセスは`root`権限で実行し、サブプロセスは通常のユーザー権限で実行できます。

```php
$server->set(array(
  'user' => 'Apache'
));
```

  * **注意**

    !> -`root`ユーザーで起動する場合にのみ有効です  
    -`user/group`構成パラメータを使用してワーカープロセスを一般ユーザーに設定した場合、ワーカープロセスで`shutdown`/[reload](/server/methods?id=reload)メソッドを使用してサービスをシャットダウンまたは再起動することはできません。`root`アカウントで`shell`端末で`kill`コマンドを実行する必要があります。
### グループ

?> **`Worker/TaskWorker`サブプロセスのプロセスユーザーグループを設定します。**【デフォルト値：スクリプトのユーザーグループ】

?> `user`構成と同じく、この構成はプロセスの所属ユーザーグループを変更して、サーバープログラムのセキュリティを向上させます。

```php
$server->set(array(
  'group' => 'www-data'
));
```

  * **注意**

    !> ルートユーザーで起動している場合にのみ有効
### chroot

?> **Redirect the file system root directory for the `Worker` process.**

?> This setting can isolate the process from the actual operating system file system for read/write operations, improving security.

```php
$server->set(array(
  'chroot' => '/data/server/'
));
```
### pid_file

**pidファイルの場所を設定します。**

`Server`が起動すると`master`プロセスの`PID`が自動的にファイルに書き込まれ、`Server`が終了すると`PID`ファイルが自動的に削除されます。

```php
$server->set(array(
    'pid_file' => __DIR__.'/server.pid',
));
```

  * **注意**

    `Server`が正常に終了しない場合、`PID`ファイルは削除されませんので、[Swoole\Process::kill($pid, 0)](/process/process?id=kill)を使用してプロセスが実際に存在するかどうかを検出する必要があります。
### buffer_input_size / input_buffer_size :id=buffer_input_size

?> **配置受信入力バッファのメモリサイズ。**【デフォルト値：`2M`】

```php
$server->set([
    'buffer_input_size' => 2 * 1024 * 1024,
]);
```
### buffer_output_size / output_buffer_size :id=buffer_output_size

?> **Configure the size of the output buffer for sending.**【Default: `2M`】

```php
$server->set([
    'buffer_output_size' => 32 * 1024 * 1024, // Must be a number
]);
```

  * **Hint**

    !> When Swoole version is >= `v4.6.7`, the default value is `UINT_MAX`, the maximum value of an unsigned INT

    * The unit is bytes, default is `2M`. Setting `32 * 1024 * 1024` means each `Server->send` can send up to `32M` bytes of data at one time
    * When invoking commands such as `Server->send`, `Http\Server->end/write`, `WebSocket\Server->push` to send data, the maximum amount of data sent at one time should not exceed the `buffer_output_size` configuration.

    !> This parameter only takes effect in [SWOOLE_PROCESS](/learn?id=swoole_process) mode, because in the PROCESS mode, data from Worker processes needs to be sent to the main process before being sent to clients, so each Worker process will have a dedicated buffer area. [Reference](/learn?id=reactor线程)
### socket_buffer_size

?> **Configure the length of the buffer for client connections.** 【Default: `2M`】

?> Different from `buffer_output_size`,  `buffer_output_size` limits the size of a `single` send by a worker process, while `socket_buffer_size` is used to set the total buffer size for communication between `Worker` and `Master` processes, see [SWOOLE_PROCESS](/learn?id=swoole_process) mode.

```php
$server->set([
    'socket_buffer_size' => 128 * 1024 *1024, //Must be a number in bytes, e.g., 128 * 1024 *1024 represents a maximum of 128MB of data waiting to be sent for each TCP client connection
]);
```

- **Data Sending Buffer**

    - When the Master process sends a large amount of data to clients, it may not be sent immediately. Instead, the data to be sent will be stored in the server's memory buffer. This parameter can adjust the size of the memory buffer.
    
    - If too much data is being sent and the buffer is filled, the server will report the following error message:
    
    ```bash
    swFactoryProcess_finish: send failed, session#1 output buffer has been overflowed.
    ```
    
    ?> If the send buffer is filled and sending fails, it will only affect the current client and not others. With a large number of TCP connections, the worst-case scenario would occupy `serv->max_connection * socket_buffer_size` bytes of memory.
    
    - Particularly for server programs involved in external communication, slow network communication may quickly fill the buffer when continuously sending data. The sent data will accumulate in the server's memory. Hence, such applications should consider network transmission capacity in their design, storing messages on disk first, and then sending new data after the client notifies that it has received them.
    
    - For example, in a video streaming service, if User `A` has a bandwidth of `100M`, sending `10M` of data within `1` second is completely feasible. User `B` may have only `1M` of bandwidth, and if `10M` of data is sent within `1` second, user `B` may take `100` seconds to receive it all. This would cause the data to accumulate in the server's memory.
    
    - Different handling strategies can be applied based on the type of data. For discardable content like video streaming, dropping some data frames in poor network conditions is acceptable. For non-discardable content like WeChat messages, storing in the server's disk first in batches of `100` messages can be done. After the user receives this batch, the server can fetch the next batch from disk and send it to the client.
### enable_unsafe_event

?> **Enable `onConnect/onClose` events.** [Default value: `false`]

?> After setting the `dispatch_mode` of Swoole to `1` or `3`, as the system cannot guarantee the order of `onConnect/onReceive/onClose`, the `onConnect/onClose` events are disabled by default;  
If the application needs `onConnect/onClose` events, and can accept the security risks that may arise from order issues, you can enable the `enable_unsafe_event` option by setting it to `true`.
### discard_timeout_request

?> **Closed connections data requests are discarded.**【Default value: `true`】

?> When `dispatch_mode` in `Swoole` configuration is set to `1` or `3`, the system cannot guarantee the order of `onConnect/onReceive/onClose`, so there may be some request data arriving in the `Worker` process after the connection is closed.

  * **Tips**

    * The `discard_timeout_request` configuration is set to `true` by default, which means that if the `worker` process receives a data request from a closed connection, it will be automatically discarded.
    * If `discard_timeout_request` is set to `false`, it means that the `Worker` process will handle data requests regardless of whether the connection is closed or not.
### enable_reuse_port

?> **Enable port reuse.**【Default value: `false`】

?> Enabling port reuse allows multiple server programs to listen on the same port.

  * **Tips**

    * `enable_reuse_port = true` to enable port reuse
    * `enable_reuse_port = false` to disable port reuse

!> Only available on Linux kernel `3.9.0` and above, and Swoole `4.5` and above.
### enable_delay_receive

?> **Setting `accept` will not automatically join the [EventLoop](/learn?id=what-is-eventloop) after the client connection.**【Default value: `false`】

?> By setting this option to `true`, `accept` will not automatically join the [EventLoop](/learn?id=what-is-eventloop) after the client connection. It will only trigger the [onConnect](/server/events?id=onconnect) callback. The `worker` process can call [$server->confirm($fd)](/server/methods?id=confirm) to confirm the connection, at which point `fd` will be added to the [EventLoop](/learn?id=what-is-eventloop) for data transmission to begin. You can also call `$server->close($fd)` to close this connection.

```php
// Enable the enable_delay_receive option
$server->set(array(
    'enable_delay_receive' => true,
));

$server->on("Connect", function ($server, $fd, $reactorId) {
    $server->after(2000, function() use ($server, $fd) {
        // Confirm the connection to start receiving data
        $server->confirm($fd);
    });
});
```
### reload_async

**設置非同期リロードスイッチ。**【デフォルト値：`true`】

設置非同期リロードスイッチ。`true`に設定すると、非同期安全なリロード機能が有効になり、`Worker`プロセスは非同期イベントの完了を待ってから終了します。詳細については[サービスの正しい再開について](/question/use?id=swoole如何正确的重启服务)を参照してください。

`reload_async`を有効にする主な目的は、サービスのリロード時に、コルーチンや非同期タスクが正常に終了することを保証することです。

```php
$server->set([
  'reload_async' => true
]);
```

  * **コルーチンモード**

    * `4.x`バージョンでは[enable_coroutine](/server/setting?id=enable_coroutine)を有効にすると、ベースシステムは追加のコルーチン数の検査を実行し、現在コルーチンが存在しない場合にプロセスが終了します。`reload_async => false`でも強制的に`reload_async`が有効になります。
### max_wait_time

?> **`Worker`プロセスが停止サービス通知を受け取った後の最大待機時間を設定します**【デフォルト値：`3`】

?> `Worker`のブロッキングによる応答の遅延による、`worker`が正常に`reload`できない場合が頻繁に発生することがあります。たとえば、コードのホットリロードを行うような製品シーンに不十分です。そのため、Swooleにはプロセスの再起動タイムアウトオプションが追加されました。詳細については、[サービスを適切に再起動する方法](/question/use?id=swoole如何正确的重启服务)を参照してください。

  * **ヒント**

    * **管理プロセスが再起動またはシャットダウンシグナルを受信するか、`max_request`に到達した場合、管理プロセスは対象の`worker`プロセスを再起動します。次のステップがあります。**

      * ロースレベルでは、指定された(`max_wait_time`)秒後にタイマーがトリガーされ、プロセスの存在をチェックします。プロセスが存在する場合、強制的に終了させ、新しいプロセスを起動します。
      * `onWorkerStop` コールバック内で後処理を行う必要があり、`max_wait_time`秒以内に後処理を完了する必要があります。
      * 対象プロセスに `SIGTERM` シグナルを順次送信し、プロセスを終了させます。

  * **注意**

    !> `v4.4.x` 以前のデフォルト値は `30` 秒です。
### tcp_fastopen

?> **TCP Fast Open機能を有効にします。**【デフォルト値：`false`】

?> この機能により、`TCP`の短い接続の応答速度が向上し、クライアントが握手の第三ステップを完了する際に`SYN`パケットにデータを搭載することができます。

```php
$server->set([
  'tcp_fastopen' => true
]);
```

  * **ヒント**

    * このパラメータはリスニングポートに設定できます。詳細を理解したい方は[Googleの論文](http://conferences.sigcomm.org/co-next/2011/papers/1569470463.pdf)をご覧ください。
### request_slowlog_file

**リクエスト遅延ログを有効にします。**`v4.4.8`バージョンから[削除されました](https://github.com/swoole/swoole-src/commit/b1a400f6cb2fba25efd2bd5142f403d0ae303366)

この遅延ログの方法は、同期ブロッキングプロセスでのみ有効であり、コルーチン環境では使用できません。Swoole4はデフォルトでコルーチンが有効になっているため、`enable_coroutine`を無効にしない限り、使用しないでください。代わりに [Swoole Tracker](https://business.swoole.com/tracker/index) のブロック検出ツールを使用してください。

有効にすると、`Manager`プロセスはタイマー信号を設定し、すべての`Task`および`Worker`プロセスを定期的に監視します。プロセスがブロックされ、リクエストが指定された時間を超過すると、プロセスの`PHP`関数呼び出しスタックが自動的に印刷されます。

バックエンドでは、`ptrace`システムコールを使用して実装されており、一部のシステムでは`ptrace`が無効になっている場合があります。遅いリクエストを追跡できない場合があります。`kernel.yama.ptrace_scope`カーネルパラメータが`0`に設定されていることを確認してください。

```php
$server->set([
  'request_slowlog_file' => '/tmp/trace.log',
]);
```

  * **タイムアウト時間**

```php
$server->set([
    'request_slowlog_timeout' => 2, // リクエストのタイムアウトを2秒に設定
    'request_slowlog_file' => '/tmp/trace.log',
]);
```

必ず書き込み権限のあるファイルである必要があります。そうでないと、ファイルの作成に失敗し、バックエンドで致命的なエラーが発生します。
### enable_coroutine

?> **非同期スタイルサーバーのコルーチンサポートを有効にしますか**

?> `enable_coroutine` を無効にすると、 [event callback functions](/server/events) 中にコルーチンが自動的に作成されなくなります。コルーチンを使用しない場合は、これを無効にするとパフォーマンスが向上します。[SwooleCoroutineとは](/coroutine)を参照してください。

  * **設定方法**
    
    * `php.ini` で `swoole.enable_coroutine = 'Off'` と設定します（[ini configuration document](/other/config.md) を参照）
    * `$server->set(['enable_coroutine' => false]);` はiniよりも優先されます

  * **`enable_coroutine` オプションの影響範囲**

      * onWorkerStart
      * onConnect
      * onOpen
      * onReceive
      * [setHandler](/redis_server?id=sethandler)
      * onPacket
      * onRequest
      * onMessage
      * onPipeMessage
      * onFinish
      * onClose
      * tick/after タイマー

!> `enable_coroutine` を有効にすると、上記のコールバック関数内で自動的にコルーチンが作成されます。

* `enable_coroutine` が `true` に設定されていると、`onRequest` コールバックで自動的にコルーチンが作成されます。 開発者は、明示的に`go`関数を使用して[コルーチンを作成する必要がありません](/coroutine/coroutine?id=create)。
* `enable_coroutine` が `false` に設定されていると、自動的にコルーチンは作成されません。開発者がコルーチンを使用したい場合は、`go` を使用してコルーチンを作成する必要があります。 コルーチン機能を使用しない場合は、Swoole1.x と同じ処理が行われます。
* ただし、この機能の有効化は、Swooleがコルーチンを使用してリクエストを処理することを意味し、イベントにブロッキング関数が含まれている場合は、事前に[ワンクリック化](/runtime)を有効にして、 `sleep`、 `mysqlnd`などのブロッキング関数または拡張機能をコルーチン化する必要があります

```php
$server = new Swoole\Http\Server("127.0.0.1", 9501);

$server->set([
    // Disable built-in coroutine
    'enable_coroutine' => false,
]);

$server->on("request", function ($request, $response) {
    if ($request->server['request_uri'] == '/coro') {
        go(function () use ($response) {
            co::sleep(0.2);
            $response->header("Content-Type", "text/plain");
            $response->end("Hello World\n");
        });
    } else {
        $response->header("Content-Type", "text/plain");
        $response->end("Hello World\n");
    }
});

$server->start();
```
### send_yield

?> **When there is not enough memory in the output buffer to send data, directly [yield](/coroutine?id=协程调度) in the current coroutine, wait for the data to be sent, and automatically [resume](/coroutine?id=协程调度) the current coroutine when the buffer is cleared to continue sending data.**【Default value: available in [dispatch_mod](/server/setting?id=dispatch_mode) 2/4 and enabled by default】

* When `Server/Client->send` returns `false` and the error code is `SW_ERROR_OUTPUT_BUFFER_OVERFLOW`, it does not return `false` to the `PHP` layer, but suspends the current coroutine with [yield](/coroutine?id=协程调度)
* `Server/Client` listens for events to check if the buffer is cleared. After this event is triggered, the data in the buffer has been sent, then [resume](/coroutine?id=协程调度) the corresponding coroutine
* After the coroutine is resumed, continue to call `Server/Client->send` to write data to the buffer. At this time, since the buffer is empty, the sending will be successful

Before improvement

```php
for ($i = 0; $i < 100; $i++) {
    // When the buffer is full, it will directly return false and report output buffer overflow
    $server->send($fd, $data_2m);
}
```

After improvement

```php
for ($i = 0; $i < 100; $i++) {
    // When the buffer is full, it will yield the current coroutine, and resume after sending is completed
    $server->send($fd, $data_2m);
}
```

!> This feature will change the default behavior of the underlying system, you can manually disable it

```php
$server->set([
    'send_yield' => false,
]);
```

  * __Affected APIs__

    * [Swoole\Server::send](/server/methods?id=send)
    * [Swoole\Http\Response::write](/http_server?id=write)
    * [Swoole\WebSocket\Server::push](/websocket_server?id=push)
    * [Swoole\Coroutine\Client::send](/coroutine_client/client?id=send)
    * [Swoole\Coroutine\Http\Client::push](/coroutine_client/http_client?id=push)
### send_timeout

`send_timeout`を設定します。 `send_yield`と一緒に使用され、指定された時間内にデータをバッファに送信できない場合、下位レベルは`false`を返し、エラーコードを`ETIMEDOUT`に設定します。エラーコードは [getLastError()](/server/methods?id=getlasterror) メソッドを使用して取得できます。

> 浮動小数点数の型であり、単位は秒で、最小粒度はミリ秒です

```php
$server->set([
    'send_yield' => true,
    'send_timeout' => 1.5, // 1.5秒
]);

for ($i = 0; $i < 100; $i++) {
    if ($server->send($fd, $data_2m) === false and $server->getLastError() == SOCKET_ETIMEDOUT) {
      echo "送信タイムアウト\n";
    }
}
```
### hook_flags

?> **Set the function scope for `one-click coroutine` Hook.**【Default value: Do not hook】

!> Available in Swoole version `v4.5+` or [4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x), see [One-click Coroutine](/runtime) for details.

```php
$server->set([
    'hook_flags' => SWOOLE_HOOK_SLEEP,
]);
```
### buffer_high_watermark

?> **Set the buffer high watermark in bytes.**

```php
$server->set([
    'buffer_high_watermark' => 8 * 1024 * 1024,
]);
``` 

外部のテキストを日本語に翻訳しましたが、コードブロック内のテキストは翻訳していません。
### buffer_low_watermark

?> **Set the buffer low watermark in bytes.**

```php
$server->set([
    'buffer_low_watermark' => 1 * 1024 * 1024,
]);
```
### tcp_user_timeout

?> TCP_USER_TIMEOUT option is a socket option at the TCP layer, representing the maximum duration, in milliseconds, that a data packet can remain unacknowledged after being sent. Please refer to the man page for more details.

```php
$server->set([
    'tcp_user_timeout' => 10 * 1000, // 10 seconds
]);
```

!> Available in Swoole version >= `v4.5.3-alpha`
### stats_file

?> **Specifies the file path where the content of [stats()](/server/methods?id=stats) is written. After setting this, a timer will be automatically set in [onWorkerStart](/server/events?id=onworkerstart) to periodically write the content of [stats()](/server/methods?id=stats) to the specified file.**

```php
$server->set([
    'stats_file' => __DIR__ . '/stats.log',
]);
```

!> Available in Swoole version >= `v4.5.5`
### event_object

?> **設定このオプション後、イベントコールバックは[オブジェクトスタイル](/server/events?id=コールバックオブジェクト)を使用します。**【デフォルト値：`false`】

```php
$server->set([
    'event_object' => true,
]);
```

!> Swooleバージョン >= `v4.6.0` で利用可能
### start_session_id

?> **セッションIDの開始値を設定します**

```php
$server->set([
    'start_session_id' => 10,
]);
```

!> Swoole version >= `v4.6.0` available
### single_thread

?> **設定をシングルスレッドにします。** これを有効にすると、ReactorスレッドはMasterプロセスのマスタースレッドと統合され、マスタースレッドで処理されます。PHP ZTS環境では、`SWOOLE_PROCESS`モードを使用する場合は、必ずこの値を`true`に設定する必要があります。

```php
$server->set([
    'single_thread' => true,
]);
```

!> Swooleのバージョン >= `v4.2.13` で利用可能
### max_queued_bytes

?> **Set the maximum queue length of the receiving buffer.** If exceeded, stop receiving.

```php
$server->set([
    'max_queued_bytes' => 1024 * 1024,
]);
```

!> Available since Swoole version `v4.5.0`
### admin_server

?> **[Swoole Dashboard](http://dashboard.swoole.com/)でサービス情報などを表示するためにadmin_serverサービスを設定します。**

```php
$server->set([
    'admin_server' => '0.0.0.0:9502',
]);
```

!> Swooleのバージョン >= `v4.8.0` で利用可能
