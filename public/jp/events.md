# イベント

このセクションでは、Swooleのすべてのコールバック関数について説明します。各コールバック関数はPHP関数に対応しており、イベントに対応しています。
## onStart

?> **`Server`のメインプロセス（master）のメインスレッドで、起動後にこの関数がコールバックされます**

```php
function onStart(Swoole\Server $server);
```

  * **パラメーター** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

* **このイベント実行前に`Server`は次の操作を行いました**

    * [Managerプロセス](/learn?id=managerプロセス)を作成して起動しました
    * [Workerサブプロセス](/learn?id=workerプロセス)を作成して起動しました
    * TCP/UDP/[UNIXソケット](/learn?id=ipcとは)ポートをすべてリッスンしましたが、接続やリクエストのAcceptはまだ開始していません
    * タイマーをリッスンしました

* **次に行う操作**

    * メイン[リアクタ](/learn?id=reactorスレッド)がイベントを受け取り始め、クライアントが`Server`に`connect`できます

**`onStart`コールバックでは、`echo`、`Log`の出力、プロセス名の変更のみが許可されています。他の操作を行ってはいけません（`server`関連の関数を呼び出すことはできません）。`onWorkerStart`と`onStart`コールバックは異なるプロセスで並行して実行され、実行順序はありません。**

`onStart`コールバック中に、`$server->master_pid`と`$server->manager_pid`の値をファイルに保存することができます。このようにすると、この2つの`PID`にシグナルを送信するスクリプトを作成して、シャットダウンや再起動の操作を実現できます。

`onStart`イベントは`Master`プロセスのメインスレッドで呼び出されます。

!> `onStart`で作成されたグローバルリソースオブジェクトは`Worker`プロセスで使用できません。なぜなら、`onStart`が呼び出される時点で`worker`プロセスがすでに作成されているからです  
新しく作成されたオブジェクトはメインプロセス内にあり、`Worker`プロセスはこのメモリ領域にアクセスできません  
そのため、グローバルオブジェクトの作成コードは`Server::start`の前に配置する必要があります。典型的な例は[Swoole\Table](/memory/table?id=完全な例)

* **セキュリティ上の注意**

`onStart`コールバックでは非同期およびコルーチンのAPIを使用できますが、これは`dispatch_func`と`package_length_func`と競合する可能性があるため、**同時に使用しないでください**。

`onStart`でタイマーを起動しないでください。コード内で`Swoole\Server::shutdown()`を実行すると、常に実行中のタイマーがあるためプログラムが終了できません。

`onStart`コールバック内では、`return`するまでサーバープログラムはクライアントの接続を受け付けませんので、同期ブロッキング関数を安全に使用できます。

* **BASEモード**

[SWOOLE_BASE](/learn?id=swoole_base)モードでは`master`プロセスがないため、`onStart`イベントは存在しません。`BASE`モードで`onStart`コールバック関数を使用しないでください。

```
警告 swReactorProcess_start: The onStart event with SWOOLE_BASE is deprecated
```
## onBeforeShutdown

?> **このイベントは`Server`が正常に終了する前に発生します**

!> Swooleバージョン >= `v4.8.0` で利用可能です。このイベント内では、コルーチンAPIを使用することができます。

```php
function onBeforeShutdown(Swoole\Server $server);
```

* **パラメーター**

    * **`Swoole\Server $server`**
        * **機能**：Swoole\Serverオブジェクト
        * **デフォルト値**：なし
        * **その他の値**：なし
## onShutdown

?> **This event occurs when the `Server` is shut down normally**

```php
function onShutdown(Swoole\Server $server);
```

  * **Parameters**

    * **`Swoole\Server $server`**
      * **Function**: Swoole\Server object
      * **Default value**: None
      * **Other values**: None

  * **`Swoole\Server` has performed the following operations before this event**

    * Closed all [Reactor](/learn?id=reactor线程) threads, `HeartbeatCheck` threads, and `UdpRecv` threads
    * Closed all `Worker` processes, [Task processes](/learn?id=taskworker进程), and [User processes](/server/methods?id=addprocess)
    * `close` all `TCP/UDP/UnixSocket` listening ports
    * Closed the main [Reactor](/learn?id=reactor线程)

  !> Forcibly `killing` the process will not trigger `onShutdown`, such as `kill -9`  
  Use `kill -15` to send the `SIGTERM` signal to the main process in order to terminate according to the normal process  
  Exiting the program with `Ctrl+C` in the command line will stop immediately, and the `onShutdown` will not be called at the low level

  * **Notes**

  !> Do not call any asynchronous or coroutine-related `API` in `onShutdown`, as all event loop facilities have been destroyed when triggering `onShutdown`;  
At this point, there is no longer a coroutine environment, and if developers need to use coroutine-related `API`, they need to manually call `Co\run` to create a [coroutine container](/coroutine?id=what-is-a-coroutine-container).
## onWorkerStart

?> **This event occurs when a Worker process/ [Task process](/learn?id=taskworker-process) starts, and objects created here can be used throughout the process lifecycle.**

```php
function onWorkerStart(Swoole\Server $server, int $workerId);
```

  * **Parameters** 
    * **`Swoole\Server $server`**
      * **Functionality**: Swoole\Server object
      * **Default**: None
      * **Other values**: None

    * **`int $workerId`**
      * **Functionality**: `Worker` process ID (not the process PID)
      * **Default**: None
      * **Other values**: None

  * `onWorkerStart/onStart` events are run concurrently without any specific order
  * You can determine whether the current process is a `Worker` process or a [Task process](/learn?id=taskworker-process) through the `$server->taskworker` property
  * When setting `worker_num` and `task_worker_num` to more than `1`, each process triggers the `onWorkerStart` event once. You can differentiate between different working processes by using [$worker_id](/server/properties?id=worker_id)
  * Tasks are sent from `worker` processes to `task` processes. After the `task` processes finish handling all tasks, they notify the `worker` processes through the [onFinish](/server/events?id=onfinish) callback function. For example, when sending notification emails to a hundred thousand users in the background, the status of the operation will be displayed as pending. You can continue with other operations, and once all emails are sent, the status will automatically change to sent.

  The following example is used to rename Worker/ [Task processes](/learn?id=taskworker-process).

```php
$server->on('WorkerStart', function ($server, $worker_id){
    global $argv;
    if($worker_id >= $server->setting['worker_num']) {
        swoole_set_process_name("php {$argv[0]} task worker");
    } else {
        swoole_set_process_name("php {$argv[0]} event worker");
    }
});
```

  If you want to achieve code reloading using the [Reload](/server/methods?id=reload) mechanism, you must `require` your business files in `onWorkerStart` instead of at the beginning of the file. Files included before the `onWorkerStart` call will not reload.

  You can place common, immutable PHP files before `onWorkerStart`. While this does not reload the code, all Workers share these files, and no additional memory is needed to store this data. Data after `onWorkerStart` needs to be stored in memory for each process.

  * `$worker_id` represents the `ID` of the `Worker` process, with the range based on [$worker_id](/server/properties?id=worker_id)
  * [$worker_id](/server/properties?id=worker_id) has no relation to the process `PID` and you can use `posix_getpid` function to get the `PID`

  * **Coroutine Support**
    * Coroutines are automatically created in the `onWorkerStart` callback function, so coroutine `API`s can be called within `onWorkerStart`

  * **Note**
    !> If a fatal error occurs or `exit` is actively called in the code, the `Worker/Task` processes will exit, and the manager process will recreate new processes. This could lead to an infinite loop of creating and destroying processes.
## onWorkerStop

?> **This event occurs when the `Worker` process is terminated. In this function, you can recycle various resources allocated by the `Worker` process.**

```php
function onWorkerStop(Swoole\Server $server, int $workerId);
```

  * **Parameters**

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default**: None
      * **Others**: None

    * **`int $workerId`**
      * **Description**: `Worker` process `id` (not the PID of the process)
      * **Default**: None
      * **Others**: None

  * **Note**

    !> -The `onWorkerStop` callback function will not be executed in case of abnormal process termination, such as when it is forcefully killed, encounters a fatal error, or has a `core dump`.  
    -Avoid calling any asynchronous or coroutine-related APIs in `onWorkerStop` because when `onWorkerStop` is triggered, all underlying [event loop](/learn?id=what-is-eventloop) facilities have already been destroyed.
## onWorkerExit

?> **This feature is only effective after enabling [reload_async](/server/setting?id=reload_async). See [How to restart the service correctly](/question/use?id=how-to-restart-the-swoole-service)**

```php
function onWorkerExit(Swoole\Server $server, int $workerId);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**：Swoole\Server object
      * **Default value**：None
      * **Other values**：None

    * **`int $workerId`**
      * **Description**：Worker process id (not PID of process)
      * **Default value**：None
      * **Other values**：None

  * **Note**

    !> - If the `Worker` process does not exit, `onWorkerExit` will continue to be triggered  
    - `onWorkerExit` is triggered within the `Worker` process, and if there is an [event loop](/learn?id=what-is-an-eventloop) in the [Task process](/learn?id=taskworker-process), it will also be triggered  
    - In `onWorkerExit`, try to remove/close asynchronous `Socket` connections as much as possible. The process will exit when the number of event handles monitored by the underlying [event loop](/learn?id=what-is-an-eventloop) is `0`  
    - When there are no event handles being monitored by the process, this function will not be called when the process ends  
    - `onWorkerStop` event callback will be executed only after the `Worker` process exits
## onConnect

?> **When a new connection comes in, it is called back in the worker process.**

```php
function onConnect(Swoole\Server $server, int $fd, int $reactorId);
```

  * **Parameters**

    * **`Swoole\Server $server`**
      * **Function**: Swoole\Server object
      * **Default Value**: None
      * **Other Values**: None

    * **`int $fd`**
      * **Function**: File descriptor of the connection
      * **Default Value**: None
      * **Other Values**: None

    * **`int $reactorId`**
      * **Function**: ID of the [Reactor](/learn?id=reactor-thread) thread where the connection is located
      * **Default Value**: None
      * **Other Values**: None

  * **Notes**

    !> `onConnect/onClose` callbacks happen in the `Worker` process, not the main process.  
    Under the `UDP` protocol, there is only the [onReceive](/server/events?id=onreceive) event, and no `onConnect/onClose` events.

    * **[dispatch_mode](/server/setting?id=dispatch_mode) = 1/3**

      * In this mode, `onConnect/onReceive/onClose` may be dispatched to different processes. PHP object data related to connections cannot be initialized in the [onConnect](/server/events?id=onconnect) callback, and data cannot be cleaned up in [onClose](/server/events?id=onclose).
      * `onConnect/onReceive/onClose` these three events may be executed concurrently, which may cause exceptions.
## onReceive

?> **データを受信したときに、この関数が`worker`プロセスでコールバックされます。**

```php
function onReceive(Swoole\Server $server, int $fd, int $reactorId, string $data);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイルディスクリプタ
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：`TCP`接続が存在する[Reactor](/learn?id=reactor糸)スレッドの`ID`
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **機能**：受信したデータ内容、テキストまたはバイナリ内容が含まれる可能性があります
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **`TCP`プロトコルのパケット完全性に関する詳細は、[TCPデータパケット境界問題] (/learn?id=tcpデータパケット境界問題)を参照してください**

    * データパケットの完全性を確保するために、`open_eof_check/open_length_check/open_http_protocol`などの下位レイヤーで提供される設定を使用できます。
    * 下位のプロトコル処理を使用しない場合は、[onReceive](/server/events?id=onreceive)後にPHPコードでデータを分析し、データパケットを結合/分割する必要があります。

    例：コード内で`$buffer = array()`を追加し、`$fd`を`key`として使用してコンテキストデータを保存できます。データが受信されるたびに文字列を連結し、`$buffer[$fd] .= $data`を実行し、その後`$buffer[$fd]`の文字列が完全なデータパケットであるかどうかを判断します。

    デフォルトでは、同一の`fd`は同一の`Worker`に割り当てられるため、データを結合できます。`dispatch_mode` = 3を使用すると、要求データは競合するため、同じ`fd`から送信されるデータが異なるプロセスに分割される可能性があるため、上記のデータパケット結合方法は使用できません。

  * **複数ポートのリッスンについては、[このセクション](/server/port)を参照してください**

    メインサーバーにプロトコルが設定されている場合、追加でリッスンされたポートはデフォルトでメインサーバーの設定を継承します。ポートのプロトコルを再設定するには、`set`メソッドを明示的に呼び出す必要があります。    

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9501);
    $port2 = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $port2->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        echo "[#".$server->worker_id."]\tClient[$fd]: $data\n";
    });
    ```

    ここでは、`on`メソッドを呼び出して[onReceive](/server/events?id=onreceive)コールバック関数を登録していますが、メインサーバーのプロトコルを上書きする`set`メソッドを呼び出していないため、新しいリッスンポートの`9502`は引き続き`HTTP`プロトコルを使用します。`telnet`クライアントを使用して`9502`ポートに文字列を送信すると、サーバーは[onReceive](/server/events?id=onreceive)をトリガーしません。

  * **注意**

    !> 自動プロトコルオプションが無効の場合、[onReceive](/server/events?id=onreceive)が1回に受信できるデータの最大は`64K`です。  
    自動プロトコル処理オプションが有効の場合、[onReceive](/server/events?id=onreceive)は完全なデータパケットを受信し、最大サイズは[package_max_length](/server/setting?id=package_max_length)を超えません。  
    バイナリ形式をサポートし、`$data`はバイナリデータかもしれません。
## onPacket

?> **`UDP`データパケットを受信したときにこの関数がコールバックされ、`worker`プロセスで発生します。**

```php
function onPacket(Swoole\Server $server, string $data, array $clientInfo);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **機能**：受信したデータ内容、テキストまたはバイナリ内容が含まれる可能性があります
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`array $clientInfo`**
      * **機能**：クライアント情報には`address/port/server_socket`など、複数のクライアント情報データが含まれます。[UDPサーバーを参照](/start/start_udp_server)
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **注意**

    !> サーバーが`TCP/UDP`ポートを同時にリッスンしている場合、`TCP`プロトコルのデータを受信すると[onReceive](/server/events?id=onreceive)が呼び出され、`UDP`データパケットを受信すると`onPacket`が呼び出されます。 サーバーで設定された`EOF`または`Length`などの自動プロトコル処理([TCPデータパケットのボーダー問題を参照](/learn?id=tcpデータパケットのボーダー問題)) は、`UDP`ポートには無効です。なぜならば、`UDP`パケット自体にメッセージの境界が存在し、追加のプロトコル処理は必要ないからです。
## onClose

?> **`TCP` client connection is closed, this function will be called in the `Worker` process.**

```php
function onClose(Swoole\Server $server, int $fd, int $reactorId);
```

* **Parameters**

    * **`Swoole\Server $server`**
        * **Functionality**: Swoole\Server object
        * **Default**: None
        * **Other values**: None

    * **`int $fd`**
        * **Functionality**: File descriptor of the connection
        * **Default**: None
        * **Other values**: None

    * **`int $reactorId`**
        * **Functionality**: Which `reactor` thread the event comes from, negative when closed by the active `close`
        * **Default**: None
        * **Other values**: None

* **Tips**

    * **Active Close**

        * When the server actively closes the connection, this parameter will be set to `-1`, and you can distinguish whether the close is initiated by the server or the client by checking `$reactorId < 0`.
        * Only when `close` method is actively called in PHP code, it is considered as an active close.

    * **Heartbeat Detection**

        * [Heartbeat detection](/server/setting?id=heartbeat_check_interval) is notified to close by the heartbeat detection thread. When closing, the `$reactorId` parameter of [onClose](/server/events?id=onclose) will not be `-1`.

* **Note**

    !> If a fatal error occurs in the [onClose](/server/events?id=onclose) callback function, it will cause connection leaks. You will see a large number of `CLOSE_WAIT` state `TCP` connections by using the `netstat` command. [Reference to Swoole video tutorial](https://course.swoole-cloud.com/course-video/4)  
    - Regardless of whether the `close` is initiated by the client or by actively calling `$server->close()` to close the connection, this event will be triggered. Therefore, this function will always be called whenever a connection is closed.  
    - In the [onClose](/server/events?id=onclose) callback, you can still call the [getClientInfo](/server/methods?id=getClientInfo) method to get connection information. The `close` to close the `TCP` connection will only be called after the [onClose](/server/events?id=onclose) callback function finishes executing.  
    - The callback to [onClose](/server/events?id=onclose) indicates that the client connection has been closed, so there is no need to execute `$server->close($fd)`. Executing `$server->close($fd)` in the code will throw a PHP error warning.
## onTask

?> **Called within the `task` process. The `worker` process can use the [task](/server/methods?id=task) function to deliver new tasks to the `task_worker` process. The current [Task process](/learn?id=taskworker-process) switches to busy state when calling the [onTask](/server/events?id=ontask) callback function, meaning it will no longer receive new tasks. Once the [onTask](/server/events?id=ontask) function returns, the process switches back to idle state and continues to receive new tasks.**

```php
function onTask(Swoole\Server $server, int $task_id, int $src_worker_id, mixed $data);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default value**: None
      * **Other values**: None

    * **`int $task_id`**
      * **Description**: ID of the `task` process that is executing the task. [The combination of `$task_id` and `$src_worker_id` is globally unique. Tasks delivered by different `worker` processes may have the same `ID`.]
      * **Default value**: None
      * **Other values**: None

    * **`int $src_worker_id`**
      * **Description**: ID of the `worker` process that delivered the task. [The combination of `$task_id` and `$src_worker_id` is globally unique. Tasks delivered by different `worker` processes may have the same `ID`.]
      * **Default value**: None
      * **Other values**: None

    * **`mixed $data`**
      * **Description**: Data content of the task
      * **Default value**: None
      * **Other values**: None

  * **Tips**

    * **Starting from v4.2.12, if [task_enable_coroutine](/server/setting?id=task_enable_coroutine) is enabled, the callback function prototype is**

      ```php
      $server->on('Task', function (Swoole\Server $server, Swoole\Server\Task $task) {
          var_dump($task);
          $task->finish([123, 'hello']); // Complete the task, end it, and return data
      });
      ```

    * **Return execution results to the `worker` process**

      * **In the [onTask](/server/events?id=ontask) function, `return` a string to indicate that the content should be returned to the `worker` process. The `worker` process will trigger the [onFinish](/server/events?id=onfinish) function, indicating that the delivered `task` has been completed. Of course, you can also trigger the [onFinish](/server/events?id=onfinish) function by using `Swoole\Server->finish()` without needing to `return`**

      * The variable returned can be any non-`null` PHP variable

  * **Notes**

    !> If a fatal error occurs during the execution of the [onTask](/server/events?id=ontask) function or it is forcefully `killed` by an external process, the current task will be discarded, but this will not affect other tasks that are currently queued.
## onFinish

?> **This callback function is called in the worker process, when the task delivered by the `worker` process is completed in the `task` process, the [task process](/learn?id=taskworker-process) will send the result of the task processing to the `worker` process through the `Swoole\Server->finish()` method.**

```php
function onFinish(Swoole\Server $server, int $task_id, mixed $data)
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default Value**: None
      * **Other values**: None

    * **`int $task_id`**
      * **Description**: ID of the `task` process executing the task
      * **Default Value**: None
      * **Other values**: None

    * **`mixed $data`**
      * **Description**: Content of the task processing result
      * **Default Value**: None
      * **Other values**: None

  * **Note**

    !> - If the `finish` method is not called in the [onTask](/server/events?id=ontask) event of the [task process](/learn?id=taskworker-process) or the result is not `return`ed, the [onFinish](/server/events?id=onfinish) event will not be triggered in the `worker` process.  
    -The `worker` process executing the [onFinish](/server/events?id=onfinish) logic is the same as the `worker` process issuing the `task` task.
## onPipeMessage

?> **When a worker process receives an [IPC](/learn?id=什么是IPC) message sent by `$server->sendMessage()`, the `onPipeMessage` event is triggered. The `worker/task` process may trigger the `onPipeMessage` event**

```php
function onPipeMessage(Swoole\Server $server, int $src_worker_id, mixed $message);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Function**：Swoole\Server object
      * **Default Value**：N/A
      * **Other Values**：N/A

    * **`int $src_worker_id`**
      * **Function**：Which `Worker` process the message is from
      * **Default Value**：N/A
      * **Other Values**：N/A

    * **`mixed $message`**
      * **Function**：Message content, can be any PHP type
      * **Default Value**：N/A
      * **Other Values**：N/A  

Japanese translation:
## onPipeMessage

?> **`$server->sendMessage()` で送信された[IPC](/learn?id=什么是IPC)メッセージをワーカープロセスが受信すると、`onPipeMessage` イベントがトリガーされます。`worker/task` プロセスは`onPipeMessage` イベントをトリガーする可能性があります**

```php
function onPipeMessage(Swoole\Server $server, int $src_worker_id, mixed $message);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $src_worker_id`**
      * **機能**：メッセージの送信元の`Worker`プロセス
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`mixed $message`**
      * **機能**：メッセージの内容、任意のPHPタイプ
      * **デフォルト値**：なし
      * **その他の値**：なし
## onWorkerError

?> **This function will be called in the `Manager` process after a `Worker/Task` process encounters an exception.**

!> This function is mainly used for alerting and monitoring. Once a Worker process exits unexpectedly, it is likely due to a fatal error or a process core dump. By recording logs or sending alert messages, developers can be notified to take appropriate actions.

```php
function onWorkerError(Swoole\Server $server, int $worker_id, int $worker_pid, int $exit_code, int $signal);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default**: None
      * **Other values**: None

    * **`int $worker_id`**
      * **Description**: ID of the exceptional `worker` process
      * **Default**: None
      * **Other values**: None

    * **`int $worker_pid`**
      * **Description**: PID of the exceptional `worker` process
      * **Default**: None
      * **Other values**: None

    * **`int $exit_code`**
      * **Description**: Exit status code, ranging from `0` to `255`
      * **Default**: None
      * **Other values**: None

    * **`int $signal`**
      * **Description**: Signal of the process exit
      * **Default**: None
      * **Other values**: None

  * **Common Errors**

    * `signal = 11`: Indicates a `segment fault` occurred in the `Worker` process, possibly triggering a low-level `BUG`. Collect `core dump` information and `valgrind` memory check logs, [report this issue to the Swoole development team](/other/issue).
    * `exit_code = 255`: Indicates a `Fatal Error` occurred in the Worker process. Check the PHP error logs, identify the problematic PHP code, and resolve the issue.
    * `signal = 9`: Indicates the `Worker` was forcefully `Kill`ed by the system. Check for any manual `kill -9` operations and verify if there is an `OOM (Out of memory)` in the `dmesg` information.
    * If an `OOM` error occurs, it may be due to excessive memory allocation. 1. Check the `Server`'s `setting` configuration, whether [socket_buffer_size](/server/setting?id=socket_buffer_size) and other allocations are too large; 2. Verify if an excessively large [Swoole\Table](/memory/table) memory module has been created.
## onManagerStart

?> **管理プロセスが開始されたときにこのイベントが発生します**

```php
function onManagerStart(Swoole\Server $server);
```

  * **ヒント**

    * このコールバック関数では管理プロセスの名前を変更できます。
    * バージョン`4.2.12`より前では、`manager`プロセスではタイマーを追加したり、タスクを投稿したり、コルーチンを使用することはできません。
    * バージョン`4.2.12`以降、`manager`プロセスでは、シグナルベースの同期モードのタイマーを使用できます。
    * `manager`プロセスでは、他のワーカープロセスにメッセージを送信するために[sendMessage](/server/methods?id=sendMessage)インターフェースを呼び出すことができます。

    * **起動順序**

      * `Task`および`Worker`プロセスはすでに作成されています
      * `Master`プロセスの状態は不明です。`Manager`と`Master`は並行しているため、`onManagerStart`コールバックが発生した時点で`Master`プロセスが準備完了しているかどうかはわかりません。

    * **BASE モード**

      * [SWOOLE_BASE](/learn?id=swoole_base)モードでは、`worker_num`、`max_request`、`task_worker_num`パラメータが設定されている場合、内部では`manager`プロセスがワーカープロセスを管理するために作成されます。そのため、`onManagerStart`と`onManagerStop`イベントコールバックが発生します。
## onManagerStop

?> **Called when the manager process is stopped**

```php
function onManagerStop(Swoole\Server $server);
```

 * **Note**

  * When `onManagerStop` is triggered, it means that the `Task` and `Worker` processes have finished running and have been reclaimed by the `Manager` process.
## onBeforeReload

?> **Workerプロセスが`Reload`される前にこのイベントが発生し、Managerプロセスでコールバックされます**

```php
function onBeforeReload(Swoole\Server $server);
```

  * **パラメータ**

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし
## onAfterReload

?> **Workerプロセスが`Reload`された後にこのイベントが発生し、Managerプロセス内でコールバックされます**

```php
function onAfterReload(Swoole\Server $server);
```

  * **パラメータ**

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし
## イベントの実行順序

* すべてのイベントコールバックは`$server->start`の後に発生します
* サーバーのシャットダウン時に最後に発生するイベントは`onShutdown`です
* サーバーが正常に起動した後、`onStart/onManagerStart/onWorkerStart`は異なるプロセス内で並行して実行されます
* `onReceive/onConnect/onClose`は`Worker`プロセスで発生します
* `Worker/Task`プロセスの起動/終了時には、それぞれ一度`onWorkerStart/onWorkerStop`が呼び出されます
* [onTask](/server/events?id=ontask)イベントは [taskプロセス](/learn?id=taskworkerプロセス)のみで発生します
* [onFinish](/server/events?id=onfinish)イベントは`worker`プロセスのみで発生します
* `onStart/onManagerStart/onWorkerStart`の`3`つのイベントの実行順序は決まっていません
## オブジェクト指向スタイル

[event_object](/server/setting?id=event_object)が有効になっていると、次のイベントコールバックがオブジェクトスタイルで使用されます。
#### [Swoole\Server\Event](/server/event_class)

* [onConnect](/server/events?id=onconnect)
* [onReceive](/server/events?id=onreceive)
* [onClose](/server/events?id=onclose)

```php
$server->on('Connect', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});

$server->on('Receive', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});

$server->on('Close', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});
```
#### [Swoole\Server\Packet](/server/packet_class)

* [onPacket](/server/events?id=onpacket)

```php
$server->on('Packet', function (Swoole\Server $serv, Swoole\Server\Packet $object) {
    var_dump($object);
});
```
#### [Swoole\Server\PipeMessage](/server/pipemessage_class)

* [onPipeMessage](/server/events?id=onpipemessage)

```php
$server->on('PipeMessage', function (Swoole\Server $serv, Swoole\Server\PipeMessage $msg) {
    var_dump($msg);
    $object = $msg->data;
    $serv->sendto($object->address, $object->port, $object->data, $object->server_socket);
});
```
#### [Swoole\Server\StatusInfo](/server/statusinfo_class)

* [onWorkerError](/server/events?id=onworkererror)

```php
$serv->on('WorkerError', function (Swoole\Server $serv, Swoole\Server\StatusInfo $info) {
    var_dump($info);
});
```
#### [Swoole\Server\Task](/server/task_class)

* [onTask](/server/events?id=ontask)

```php
$server->on('Task', function (Swoole\Server $serv, Swoole\Server\Task $task) {
    var_dump($task);
});
```
#### [Swoole\Server\TaskResult](/server/taskresult_class)

* [onFinish](/server/events?id=onfinish)

```php
$server->on('Finish', function (Swoole\Server $serv, Swoole\Server\TaskResult $result) {
    var_dump($result);
});
```
