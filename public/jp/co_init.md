# サーバー（コルーチンスタイル） <!-- {docsify-ignore-all} -->

`Swoole\Coroutine\Server` は[非同期スタイル](/server/init)のサーバーと異なる点は、`Swoole\Coroutine\Server` が完全にコルーチン化されたサーバーであることです。[完全な例](/coroutine/server?id=完整示例)を参照してください。

## 利点：

- イベントコールバック関数を設定する必要がありません。接続の確立、データの受信、データの送信、接続のクローズはすべて順番に行われ、[非同期スタイル](/server/init)の並行性の問題は存在しません。例：

```php
$serv = new Swoole\Server("127.0.0.1", 9501);

//接続受け入れイベントを監視
$serv->on('Connect', function ($serv, $fd) {
    $redis = new Redis();
    $redis->connect("127.0.0.1",6379);//ここでOnConnectのコルーチンはサスペンドされる
    Co::sleep(5);//ここでconnectが遅い状況を模倣するためにsleep
    $redis->set($fd,"fd $fd connected");
});

//データ受信イベントを監視
$serv->on('Receive', function ($serv, $fd, $reactor_id, $data) {
    $redis = new Redis();
    $redis->connect("127.0.0.1",6379);//ここでonReceiveのコルーチンはサスペンドされる
    var_dump($redis->get($fd));//onReceiveのコルーチンのredis接続が先に確立される可能性があり、上記のsetがまだ実行されていないため、ここでgetはfalseになり、論理エラーが発生します
});

//接続クローズイベントを監視
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

//サーバーを起動
$serv->start();
```

上記の`非同期スタイル`のサーバーは、イベントの順序を保証できないため、`onConnect`が完了する前に`onReceive`に入ることを保証できません。なぜなら、コルーチン化を有効にした場合、`onConnect`と`onReceive`のコールバックは両方とも自動的にコルーチンを作成し、IOに遭遇すると[コルーチンスケジューリング](/coroutine?id=协程调度)が発生するため、非同期スタイルではスケジュール順序が保証できず、一方、コルーチンスタイルのサーバーにはこの問題がありません。

- サービスを動的に開始/終了できます。非同期スタイルのサービスは`start()`が呼び出された後は何もできませんが、コルーチンスタイルはサービスを動的に開始/終了できます。

## 欠点：

- コルーチンスタイルのサービスは自動的に複数のプロセスを作成しません。複数コアを活用するには[Process\Pool](/process/process_pool)モジュールを組み合わせて使用する必要があります。
- コルーチンスタイルのサービスは実際には[Co\Socket](/coroutine_client/socket)モジュールのラッパーであり、したがって、コルーチンスタイルを使用するにはソケットプログラミングの経験が必要です。
- 現在、非同期スタイルのサーバーほど高いラッピングレベルがないため、一部の機能は手動で実装する必要があります。たとえば、`reload`機能は自分でシグナルを監視してロジックを実装する必要があります。
