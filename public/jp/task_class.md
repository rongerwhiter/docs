# Swoole\Server\Task

`Swoole\Server\Task`の詳細についてここに説明があります。このクラスは非常に単純ですが、`new Swoole\Server\Task()`を使用して`Task`オブジェクトを取得することはできません。このようなオブジェクトには完全にサーバー関連の情報が含まれておらず、`Swoole\Server\Task`の任意のメソッドを実行すると致命的なエラーが発生します。

```shell
Invalid instance of Swoole\Server\Task in /home/task.php on line 3
```


## プロパティ

### $data

`worker`プロセスから`task`プロセスに渡されたデータ`data`であり、このプロパティは`string`タイプの文字列です。

```php
Swoole\Server\Task->data
```

### $dispatch_time

データが`task`プロセスに到着する時間`dispatch_time`を返します。このプロパティは`double`タイプです。

```php
Swoole\Server\Task->dispatch_time
```

### $id

データが`task`プロセスに到着する時間`dispatch_time`を返します。このプロパティは`int`タイプの整数です。

```php
Swoole\Server\Task->id
```

### $worker_id

データがどの`worker`プロセスから来たかを返します。このプロパティは`int`タイプの整数です。

```php
Swoole\Server\Task->worker_id
```

### $flags

非同期タスクのいくつかのフラグ情報`flags`を示します。このプロパティは`int`タイプの整数です。

```php
Swoole\Server\Task->flags
```

?> `flags`の返り値には、以下の種類があります：  
  - SWOOLE_TASK_NOREPLY | SWOOLE_TASK_NONBLOCK これは`Worker`プロセスから`task`プロセスに送信されたものではないことを示します。この場合、`onTask`イベントで`Swoole\Server::finish()`を呼び出すと警告が発生します。  
  - SWOOLE_TASK_CALLBACK | SWOOLE_TASK_NONBLOCK `Swoole\Server::finish()`で最後のコールバック関数がnullでない場合、`onFinish`イベントは実行されず、代わりにこのコールバック関数のみが実行されます。 
  - SWOOLE_TASK_COROUTINE | SWOOLE_TASK_NONBLOCK タスクをコルーチン形式で処理することを示します。 
  - SW_TASK_NONBLOCK デフォルト値で、上記の3つの状況がすべて該当しない場合。

## メソッド

### finish()

[Taskプロセス](/learn?id=taskworkerプロセス)内で、投げられたタスクが完了したことを`Worker`プロセスに通知するために使用されます。この関数は結果データを`Worker`プロセスに渡すことができます。

```php
Swoole\Server\Task->finish(mixed $data): bool
```

  * **引数**

    * `mixed $data`

      * 機能：タスク処理の結果の内容
      * デフォルト値：なし
      * 他の値：なし

  * **ヒント**
    * `finish`メソッドは複数回呼び出すことができ、`Worker`プロセスは複数回[onFinish](/server/events?id=onfinish)イベントを発生させます
    * [onTask](/server/events?id=ontask)コールバック関数内で`finish`メソッドを呼び出した後、`return`したデータも引き続き[onFinish](/server/events?id=onfinish)イベントをトリガーします
    * `Swoole\Server\Task->finish`はオプションです。`Worker`プロセスがタスクの実行結果に興味を持たない場合は、この関数を呼び出す必要はありません
    * [onTask](/server/events?id=ontask)コールバック関数内で`return`文字列を使用すると、`finish`を呼び出したのと同じ効果があります

  * **注意**

  !> `Swoole\Server\Task->finish`関数を使用するには`Server`に[onFinish](/server/events?id=onfinish)コールバック関数を設定する必要があります。この関数は[Taskプロセス](/learn?id=taskworkerプロセス)の[onTask](/server/events?id=ontask)コールバックでのみ使用できます


### pack()

指定されたデータをシリアライズします。

```php
Swoole\Server\Task->pack(mixed $data): string|false
```

  * **引数**

    * `mixed $data`

      * 機能：タスク処理の結果内容
      * デフォルト値：なし
      * 他の値：なし

  * **戻り値**
    * 成功すると、シリアライズ後の結果が返されます。 

### unpack()

指定されたデータをデシリアライズします。

```php
Swoole\Server\Task->unpack(string $data): mixed
```

  * **引数**

    * `string $data`

      * 機能：デシリアライズする必要があるデータ
      * デフォルト値：なし
      * 他の値：なし

  * **戻り値**
    * 成功すると、デシリアライズ後の結果が返されます。 

## 使用例
```php
<?php
$server->on('task', function(Swoole\Server $serv, Swoole\Server\Task $task) {
    $task->finish(['result' => true]);
});
``` 
