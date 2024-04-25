# タスクの実行

サーバープログラムで、例えばチャットサーバーがブロードキャストを送信したり、Webサーバーがメールを送信したりなど、時間のかかる操作を行う必要がある場合があります。これらの関数を直接実行すると、現在のプロセスがブロックされ、サーバーの応答が遅くなります。

Swooleでは、非同期タスク処理の機能が提供されており、タスクワーカープールに非同期タスクを投稿して実行することができます。これにより、現在のリクエストの処理速度に影響を与えることなく、時間のかかるタスクを実行できます。

## プログラムコード

最初のTCPサーバーをベースに、[onTask](/server/events?id=ontask)および[onFinish](/server/events?id=onfinish)という2つのイベントコールバック関数を追加するだけで、非同期タスク処理を追加できます。また、タスクプロセスの数を設定する必要があります。タスクの実行時間とタスク量に合わせて、適切なタスクプロセス数を設定してください。

以下のコードを`task.php`に記述してください。

```php
$serv = new Swoole\Server('127.0.0.1', 9501);

// 非同期タスクの作業プロセス数を設定する。
$serv->set([
    'task_worker_num' => 4
]);

// Workerプロセス内で実行されるこのコールバック関数。
$serv->on('Receive', function($serv, $fd, $reactor_id, $data) {
    // 非同期タスクを投稿する
    $task_id = $serv->task($data);
    echo "Dispatch AsyncTask: id={$task_id}\n";
});

// 非同期タスクを処理する(このコールバック関数はタスクプロセス内で実行される)。
$serv->on('Task', function ($serv, $task_id, $reactor_id, $data) {
    echo "New AsyncTask[id={$task_id}]".PHP_EOL;
    // タスクの実行結果を返す
    $serv->finish("{$data} -> OK");
});

// 非同期タスクの結果を処理する(このコールバック関数はWorkerプロセスで実行される)。
$serv->on('Finish', function ($serv, $task_id, $data) {
    echo "AsyncTask[{$task_id}] Finish: {$data}".PHP_EOL;
});

$serv->start();
```

`$serv->task()`を呼び出した後、プログラムはすぐに返り、コードの次の行に進みます。`onTask`コールバック関数はタスクプール内で非同期に実行されます。実行が完了した後、`$serv->finish()`を呼び出して結果を返します。

!> finish操作はオプションです。結果を返さずに終了することもできます。`onTask`イベントで`return`を使用して結果を返す場合、`Swoole\Server::finish()`操作が呼び出されることになります。
