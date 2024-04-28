# Basic Knowledge
## 4つのコールバック関数の設定方法

* **無名関数**

```php
$server->on('Request', function ($req, $resp) use ($a, $b, $c) {
    echo "hello world";
});
```
!> `use`を使って無名関数にパラメータを渡すことができます

* **クラスの静的メソッド**

```php
class A
{
    static function test($req, $resp)
    {
        echo "hello world";
    }
}
$server->on('Request', 'A::Test');
$server->on('Request', array('A', 'Test'));
```
!> 対応する静的メソッドは`public`である必要があります

* **関数**

```php
function my_onRequest($req, $resp)
{
    echo "hello world";
}
$server->on('Request', 'my_onRequest');
```

* **オブジェクトのメソッド**

```php
class A
{
    function test($req, $resp)
    {
        echo "hello world";
    }
}

$object = new A();
$server->on('Request', array($object, 'test'));
```

!> 対応するメソッドは`public`である必要があります
## 同期/非同期IO

`Swoole4+`では、すべてのビジネスコードは同期的な書き方です（`Swoole1.x`時代に非同期の書き方もサポートされましたが、現在は非同期クライアントが削除され、コルーチンクライアントを使用して同等の要件を満たすことができます）。これにより、精神的な負担がないため、人間の思考習慣に合っていますが、同期的な書き方は非同期IO/同期IOを持つ可能性があります。

同期IOまたは非同期IOのいずれであっても、`Swoole/Server`は大量の`TCP`クライアント接続を維持できます（[SWOOLE_PROCESSモード](/learn?id=swoole_process)を参照）。あなたのサービスがブロッキングかノンブロッキングかは特定のパラメータを個別に設定する必要はありませんが、あなたのコードで同期IOの操作がどれだけ含まれているかに依存します。

**同期IOとは：**
 
簡単な例は`MySQL->query`を実行する時です。このプロセスはMySQLの結果を待ち、結果を返した後にコードを継続します。そのため、同期IOのサービスの並行処理能力は非常に低いです。

**どのようなコードが同期IOに該当するか：**

- [コルーチンの一括有効化](/runtime)がオフの場合、コード内のほとんどのIO関連の操作は同期IOになります。コルーチン化された後は非同期IOに変わり、プロセスは単純に待機しません。[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)を参照してください。
- 一部のIOは一括でコルーチン化できず、同期IOを非同期IOに変換できない場合があります。例えば`MongoDB`（`Swoole`がこの問題を解決することを信じています）。コードを書くときはこれに注意してください。

!> [コルーチン](/coroutine) は並行性を高めるためのものです。アプリケーションが高並行性を持っていない場合、または特定の非同期IO化できない操作（上記のMongoDBなど）を実行する必要がある場合、[コルーチンを有効化](/runtime)しなくても完全に大丈夫です。[enable_coroutine](/server/setting?id=enable_coroutine)を無効にし、いくつかの`Worker`プロセスを起動しても構いません。これにより、`Fpm/Apache`と同じモデルになります。`Swoole`は[長時間稼働](https://course.swoole-cloud.com/course-video/80)のプロセスであるため、同期IOでもパフォーマンスが大幅に向上し、実際のアプリケーションでは多くの企業がこのようにしています。
### 同期IOを非同期IOに変換する

[前のセクション](/learn?id=同期io异步io)では、同期/非同期IOについて説明しました。 `Swoole`の下では、いくつかの同期`IO`操作を非同期IOに変換することができます。

- [ワンクリックコルーチン化](/runtime)を有効にすると、`MySQL`、`Redis`、`Curl`などの操作が非同期IOに変換されます。
- [Event](/event)モジュールを使用してイベントを手動で管理し、fdを[EventLoop](/learn?id=什么是eventloop)に追加して非同期IOに変換する例：

```php
//ファイル変更を監視するinotifyの利用
$fd = inotify_init();
// $fdをSwooleのEventLoopに追加
Swoole\Event::add($fd, function () use ($fd){
    $var = inotify_read($fd);//ファイルが変更された後、ファイルの変更を読み取る。
    var_dump($var);
});
```

上記のコードでは、`Swoole\Event::add`を呼び出さないとIOが非同期化されず、`inotify_read()`がWorkerプロセスをブロックし、他のリクエストは処理されません。

- `Swoole\Server`の[sendMessage()](/server/methods?id=sendMessage)メソッドを使用してプロセス間通信を行う場合、デフォルトでは`sendMessage`は同期IOですが、ある状況では`Swoole`がそれを非同期IOに変換します。[Userプロセス](/server/methods?id=addprocess)の例：

```php
$serv = new Swoole\Server("0.0.0.0", 9501, SWOOLE_BASE);
$serv->set(
    [
        'worker_num' => 1,
    ]
);

$serv->on('pipeMessage', function ($serv, $src_worker_id, $data) {
    echo "#{$serv->worker_id} message from #$src_worker_id: $data\n";
    sleep(10);//sendMessageからのデータを受け取らないと、バッファがすぐにいっぱいになります
});

$serv->on('receive', function (swoole_server $serv, $fd, $reactor_id, $data) {

});

//Case 1: 同期IO（デフォルトの動作）
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//デフォルトでは、バッファがいっぱいになると、ここでブロックします
    }
}, false);

//Case 2: enable_coroutineパラメータを使用してUserProcessプロセスのコルーチンサポートを有効にします。他のコルーチンがEventLoopのスケジュールを逃すことを防ぐために、
//SwooleはsendMessageを非同期IOに変換します
$enable_coroutine = true;
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファがいっぱいになると、プロセスはブロックされず、エラーが発生します
    }
}, false, 1, $enable_coroutine);

//Case 3: UserProcessプロセス内で非同期コールバック（たとえばタイマーの設定、Swoole\Event::addなど）が設定されている場合、
//他のコールバック関数がEventLoopのスケジュールから逃すことを防ぐために、SwooleはsendMessageを非同期IOに変換します
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    swoole_timer_tick(2000, function ($interval) use ($worker, $serv) {
        echo "timer\n";
    });
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファがいっぱいになると、プロセスはブロックされず、エラーが発生します
    }
}, false);

$serv->addProcess($userProcess);

$serv->start();
```

- 同様に、[Taskプロセス](/learn?id=taskworker进程)は`sendMessage()`を使用してプロセス間通信を行いますが、Serverの[task_enable_coroutine](/server/setting?id=task_enable_coroutine)設定でコルーチンサポートを有効にし、`Case 3`は存在しません。つまり、タスクプロセスは非同期コールバックを有効にすることでsendMessageを非同期IOに変換しません。
## EventLoopとは何ですか

`EventLoop`とは、イベントループと呼ばれ、単純には`epoll_wait`と理解されます。イベントループは、すべてのイベントが発生する可能性があるハンドル（fd）を`epoll_wait`に追加し、これらのイベントには読み取り可能、書き込み可能、エラーなどが含まれます。

対応するプロセスは、`epoll_wait`というカーネル関数でブロックされ、イベントが発生した（またはタイムアウトした）場合、`epoll_wait`関数はブロックが解除されて結果を返し、対応するPHP関数がコールバックされます。たとえば、クライアントからデータを受信した場合、`onReceive`コールバック関数が呼び出されます。

`epoll_wait`に多くのfdが追加され、同時に多くのイベントが発生した場合、`epoll_wait`関数が戻ると、それぞれのコールバック関数が順番に呼び出され、これを一つのイベントループと呼びます。これは、IOマルチプレクシングと呼ばれ、次のイベントループのために再度ブロックされ`epoll_wait`が呼び出されます。
## TCP packet boundary problem

在单线程的情况下[快速启动中的代码](/start/start_tcp_server)可以正常运行，但是一旦并发量增加，就会出现TCP数据包边界问题。`TCP`协议在底层解决了`UDP`协议的顺序和重传丢失问题，但是却引入了新的问题，即`TCP`协议是流式的，数据包没有明确的分界线，因此应用程序在使用`TCP`通信时会遇到这些难题，俗称TCP粘包问题。

由于`TCP`通信是流式的，当接收一个大数据包时，它可能会被拆分成多个小数据包进行发送。多次`Send`操作底层也可能会合并成一次发送。为了解决这个问题，需要执行以下两种操作：

* 分包：`Server`接收到多个数据包后，需要将它们拆分成单独的数据包
* 合包：`Server`接收到的数据可能只是包的一部分，需要缓存数据并将其合并成完整的数据包

因此，在进行TCP网络通信时需要制定通信协议。常见的TCP通用网络通信协议包括`HTTP`、`HTTPS`、`FTP`、`SMTP`、`POP3`、`IMAP`、`SSH`、`Redis`、`Memcache`、`MySQL`等。

值得一提的是，Swoole内置了许多常见通用协议的解析，以解决这些协议在服务器端出现的TCP数据包边界问题，只需要简单的配置即可，参考[open_http_protocol](/server/setting?id=open_http_protocol)/[open_http2_protocol](/http_server?id=open_http2_protocol)/[open_websocket_protocol](/server/setting?id=open_websocket_protocol)/[open_mqtt_protocol](/server/setting?id=open_mqtt_protocol)。

除了通用协议之外，还可以自定义协议，`Swoole`支持两种类型的自定义网络通信协议。

* **EOF结束符协议**

`EOF`协议的处理原理是在每个数据包的末尾追加一组特殊字符来表示包已经结束。例如像`Memcache`、`FTP`、`SMTP`都使用`\r\n`作为结束符。发送数据时只需在包的末尾加上`\r\n`即可。使用`EOF`协议处理时，必须确保数据包中间不会出现`EOF`，否则就会导致错误的拆包。

在`Server`和`Client`的代码中只需设置两个参数即可使用`EOF`协议处理。

```php
$server->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
```

但是上述`EOF`的配置性能会较差，因为Swoole需要遍历每个字节以查看数据是否为`\r\n`。除了上面的方式之外，也可以采用下面这种示例进行设置。

```php
$server->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
```

这种配置的性能要更好一些，不需要遍历数据，但它只能解决`拆包`问题，无法解决`合包`问题。也就是说，有可能`onReceive`一次接收到来自客户端的多个请求，需要手动进行拆包，例如使用`explode("\r\n", $data)`。而这种配置的主要用途是针对请求-响应式的服务（例如终端输入命令），无需考虑拆分数据的问题。原因是客户端发送一个请求后，必须等待服务器端返回当前请求的响应数据，才会发送第二个请求，不会同时发送两个请求。

* **固定包头+包体协议**

固定包头的方法非常通用，在服务器端程序中经常能看到。这种协议的特点是，一个数据包总是由包头+包体两部分组成。包头通常使用一个字段来指定包体或整个包的长度，长度通常使用`2`字节或`4`字节的整数来表示。服务器收到包头后，可以根据长度值精确控制需要接收多少数据才能构成完整的数据包。`Swoole`的配置可以很好地支持这种协议，可以通过灵活设置`4`个参数来适应各种情况。

`Server`在[onReceive](/server/events?id=onreceive)回调函数中处理数据包，设置了协议处理后，只有在收到完整的数据包时才会触发[onReceive](/server/events?id=onreceive)事件。在设置了协议处理后，客户端调用 [$client->recv()](/client?id=recv) 不需要再传递长度参数，`recv`函数在接收到完整的数据包或出现错误后会返回。

```php
$server->set(array(
    'open_length_check' => true,
    'package_max_length' => 81920,
    'package_length_type' => 'n', //see php pack()
    'package_length_offset' => 0,
    'package_body_offset' => 2,
));
```

!> 每个配置的具体含义请参考`服务端/客户端`部分的[设置](/server/setting?id=open_length_check)小节。
## IPCとは

同じホスト上の2つのプロセス間での通信（IPC）には、Swooleでは`Unix Socket`と`sysvmsg`の2つの方法が使用されています。以下ではそれぞれを紹介します：

- **Unix Socket**  

    `UNIX Domain Socket`の完全な名前は、`UDS`とも呼ばれ、ソケットAPI（socket、bind、listen、connect、read、write、closeなど）を使用します。TCP/IPとは異なり、IPやポートを指定する必要はなく、代わりにファイル名を使用して表します（例えばFPMとNginxの間の`/tmp/php-fcgi.sock`）。UDSはLinuxカーネルで実装されたメモリ全体を使った通信で、何も入出力の消耗がない。1つのプロセスが`write`し、もう1つのプロセスが`read`する場合、1024バイトのデータを1回読み書きしたテストでは、100万回の通信が1.02秒しかかかりませんでした。また、非常に強力な機能を持っており、`Swoole`ではデフォルトでこのIPC方法が使用されています。   
      
    * **`SOCK_STREAM`と`SOCK_DGRAM`**

        - `Swoole`では、`UDS`通信には`SOCK_STREAM`と`SOCK_DGRAM`の2つのタイプがあります。これはTCPとUDPの違いと理解できます。`SOCK_STREAM`タイプを使用する場合は、同様に[TCPデータパケットの境界問題](/learn?id=tcp数据包边界问题)に注意する必要があります。   
        - `SOCK_DGRAM`タイプを使用する場合、TCPデータパケットの境界問題を考慮する必要がなく、`send()`のデータはすべて境界があり、送信されるデータは受信時に受信され、パケットの転送中のパケットの損失や順序の問題はありません。`send`で書き込んだ順序と`recv`で読み取る順序は完全に一致します。`send`が成功した場合は必ず`recv`することができます。

    データの送信量が小さいIPCでは`SOCK_DGRAM`の方法が非常に適しています。**64kの制限を持つIPパケットがあるため、IPC時には1度の送信データが64kを超えてはいけないことに注意する必要があり、また受信速度が遅いと操作システムのバッファがいっぱいになりパケットが破棄される可能性があることにも注意する必要があります。UDPはパケットの破棄を許可しているため、バッファを適切に大きくすることができます**。

- **sysvmsg**
     
    これはLinuxが提供する`メッセージキュー`で、このIPC方法は通信にキーとしてファイル名を使用します。この方法は非常に柔軟性に欠けており、実際のプロジェクトではあまり使用されていませんので、詳細は省略します。

    * **このIPC方法は次の2つのシナリオで有用です:**

        - データの損失を防ぐために、サービス全体がクラッシュした場合でも、再起動されたらキュー内のメッセージが残り、引き続き消費できますが、**汚れたデータの問題もあります**。
        - データを外部に配信することができます。Swooleの`Workerプロセス`が`Taskプロセス`にタスクを投入するためにメッセージキューを使用したり、サードパーティのプロセスがタスクをキューに追加してTaskが消費することができたり、コマンドラインで手動でメッセージをキューに追加することもできます。
## Masterプロセス、Reactorスレッド、Workerプロセス、Taskプロセス、Managerプロセスの違いと関係 :id=diff-process
### Masterプロセス

* Masterプロセスはマルチスレッドプロセスであり、[プロセス/スレッド構造図](/server/init?id=プロセススレッド構造図)を参照してください。
### Reactorスレッド

* Reactorスレッドはマスタープロセス内で作成されるスレッドです
* クライアントの`TCP`接続の管理、ネットワーク`IO`の処理、プロトコルの処理、データの送受信を担当します
* どんなPHPコードも実行しません
* `TCP`クライアントからのデータをバッファに入れ、結合し、完全なリクエストデータパケットに分割します
### Workerプロセス

- `Reactor`スレッドから受信したリクエストデータパケットを受け取り、`PHP`コールバック関数を実行してデータを処理します
- レスポンスデータを生成し、それを`Reactor`スレッドに転送し、`Reactor`スレッドが`TCP`クライアントに送信します
- 非同期ノンブロッキングモードまたは同期ブロッキングモードで実行できます
- `Worker`は複数プロセスで実行されます
### TaskWorker プロセス

- `Worker` プロセスから Swoole\Server -> [task](/server/methods?id=task)/[taskwait](/server/methods?id=taskwait)/[taskCo](/server/methods?id=taskCo)/[taskWaitMulti) メソッドを介して受け取ったタスクを処理します。
- タスクを処理し、結果データを`Worker`プロセスに返します（[Swoole\Server->finish](/server/methods?id=finish)を使用）。
- 完全に**同期ブロッキング**モードです。
- `TaskWorker` は複数プロセスで実行され、[taskの完全な例](/start/start_task)も参照してください。
### Managerプロセス

* `worker`/`task`プロセスの作成/回収を担当します

彼らの関係は、`Reactor`が`nginx`であり、`Worker`が`PHP-FPM`であると考えることができます。`Reactor`スレッドは非同期並行でネットワークリクエストを処理し、それを`Worker`プロセスに転送して処理します。`Reactor`と`Worker`の間は[unixSocket](/learn?id=什么是IPC)を介して通信します。

`PHP-FPM`アプリケーションでは、タスクを`Redis`などのキューに非同期で投稿し、バックグラウンドでいくつかの`PHP`プロセスを起動してこれらのタスクを非同期で処理することがよくあります。`Swoole`が提供する`TaskWorker`は、タスクの投稿、キュー、`PHP`タスクの処理プロセス管理を一体化したより包括的なソリューションです。提供される低レベルの`API`を使用すると、非常に簡単に非同期タスクの処理を実装できます。さらに、`TaskWorker`はタスクの完了後に結果を`Worker`にフィードバックできます。

`Swoole`の`Reactor`、`Worker`、`TaskWorker`は密接に結びついており、より高度な使用方法を提供します。

より一般的な比喩として、`Server`が工場であるとすれば、`Reactor`は営業部門であり、顧客の注文を受け付けます。そして`Worker`は労働者であり、営業が注文を受けると、`Worker`は必要なものを生産します。そして`TaskWorker`は管理者、雑用を手伝い、`Worker`が集中して作業できるようにします。

図：

![process_demo](_images/server/process_demo.png)
## Serverの2つの実行モードについて

`Swoole\Server`のコンストラクタの第3引数には、[SWOOLE_BASE](/learn?id=swoole_base)または[SWOOLE_PROCESS](/learn?id=swoole_process)という2つの定数を指定できます。以下では、これら2つのモードの違い、利点、欠点について説明します。
### SWOOLE_PROCESS

`Server`のSWOOLE_PROCESSモードでは、すべてのクライアントのTCP接続が[メインプロセス](/learn?id=reactor線の)と確立されます。内部実装はかなり複雑であり、多くのプロセス間通信とプロセス管理メカニズムが使用されています。極めて複雑なビジネスロジックのシナリオに適しています。`Swoole`は、完全なプロセス管理とメモリ保護メカニズムを提供しています。
非常に複雑なビジネスロジックの場合でも、安定した長時間稼働が可能です。

`Swoole`は[Reactor](/learn?id=reactor線)スレッド内で`Buffer`機能を提供しており、多数の遅い接続やバイト単位での悪意のあるクライアントに対処できます。
### プロセスモデルの利点：

- 接続とデータリクエストの送信が分離されているため、ある接続が大量のデータを持ち、別の接続が小さなデータを持っている場合でも、`Worker`プロセスが均等にならないようになります。
- `Worker`プロセスで致命的なエラーが発生しても、接続は切断されません。
- シングルコネクション並行処理を実現し、少数の`TCP`接続のみを維持し、リクエストを複数の`Worker`プロセスで並行して処理できます。
#### プロセスモデルの欠点：

* `master`プロセスと`worker`プロセス間での通信に`unixSocket`を使用するため、`IPC`が`2`回発生する。
* `SWOOLE_PROCESS`はPHP ZTSをサポートしていません。この場合、`SWOOLE_BASE`を使用するか、[single_thread](/server/setting?id=single_thread)をtrueに設定する必要があります。
### SWOOLE_BASE

SWOOLE_BASEは、従来の非同期非ブロッキング`Server`です。`Nginx`や`Node.js`などのプログラムと完全に同じです。

[worker_num](/server/setting?id=worker_num)パラメータは`BASE`モードでも有効であり、複数の`Worker`プロセスが起動します。

TCP接続リクエストが到着すると、すべてのWorkerプロセスがこの1つの接続を競合し、最終的に1つのworkerプロセスが成功してクライアントと直接TCP接続を確立します。その後、この接続のすべてのデータの送受信は、このworkerとの通信を介して行われ、メインプロセスのReactorスレッドを経由しません。

* `BASE` モードでは、`Master`プロセスの役割はなく、[Manager](/learn?id=manager-proccess)プロセスの役割のみが存在します。
* 各 `Worker` プロセスは同時に`SWOOLE_PROCESS`モードでの[Reactor](/learn?id=reactor-thread)スレッドと `Worker` プロセスの2つの役割を担います。
* `BASE` モードでは `Manager` プロセスはオプションです。`worker_num=1`が設定され、かつ `Task` および `MaxRequest` 機能が使用されていない場合、基本層は直接個別の`Worker`プロセスを作成し、`Manager`プロセスを作成しません。
#### BASEモデルの利点：

* `BASE`モデルには`IPC`のオーバーヘッドがなく、パフォーマンスが向上します
* `BASE`モデルのコードはよりシンプルで、エラーが少ないです
#### BASEモードの欠点：

* `TCP`接続は`Worker`プロセスで維持されるため、特定の`Worker`プロセスがダウンすると、その`Worker`内のすべての接続が閉じられます
* 少数の`TCP`長期接続はすべての`Worker`プロセスを利用できません
* `TCP`接続は`Worker`にバインドされているため、長期接続では、一部の接続のデータ量が多いと、その接続がある`Worker`プロセスの負荷が非常に高くなります。しかし、一部の接続のデータ量が少ない場合、その接続がある`Worker`プロセスの負荷は非常に低くなります。異なる`Worker`プロセス間で均等に負荷を分散することはできません。
* コールバック関数にブロッキング操作があると、`Server`が同期モードに退化し、この状態でTCPの[バックログ](/server/setting?id=backlog)キューがいっぱいになる可能性があります。
#### BASEモードの利用シーン：

クライアント間で相互作用が必要ない場合には、`BASE`モードを使用できます。例えば、`Memcache`、`HTTP`サーバーなどです。
#### BASEモードの制限：

`BASE`モードでは、[Serverメソッド](/server/methods)において、[send](/server/methods?id=send)と[close](/server/methods?id=close)以外のメソッドは**他のプロセスとの実行をサポートしません**。

!> v4.5.xでは`BASE`モードで`send`メソッドのみが他のプロセスとの実行をサポートしています。v4.6.xでは`send`および`close`メソッドがサポートされています。
この3つのクラスの違いは何ですか？それぞれのクラスの役割や特性について教えてください。
### プロセス

[プロセス](/process/process) は、Swoole が提供するプロセス管理モジュールであり、PHP の `pcntl` を代替するために使用されます。

- プロセス間通信を簡単に実装できます。
- 標準入力と出力をリダイレクトできるため、子プロセス内で `echo` を使用しても画面に出力されず、パイプに書き込まれ、キーボード入力はパイプからデータを読み取ることができます。
- [exec](/process/process?id=exec) インターフェースを提供し、作成したプロセスは他のプログラムを実行でき、元の `PHP` 親プロセスと間で簡単に通信できます。

!> コルーチン環境では `Process` モジュールを使用できません。`runtime hook`+`proc_open` を使用して実装できます。詳細は[こちら](/coroutine/proc_open)を参照してください。
### Process\Pool

[Process\Pool](/process/process_pool)は、サーバーのプロセス管理モジュールをPHPクラスにカプセル化し、PHPコードでSwooleのプロセスマネージャを使用できるようにします。

実際のプロジェクトでは、`Redis`、`Kafka`、`RabbitMQ`などを利用したマルチプロセスキューコンシューマーやマルチプロセスウェブクローラーなど、長期間実行されるスクリプトを作成する必要がよくあります。これらの場合、開発者は`pcntl`および`posix`関連の拡張機能を使用してマルチプロセスプログラミングを実装する必要がありますが、Linuxシステムプログラミングの豊富な知識も必要です。そうでないと問題が発生する可能性が高くなります。Swooleのプロセス管理機能を使用すると、多重プロセススクリプトプログラミング作業が大幅に簡素化されます。

- ワーカープロセスの安定性を確保する
- シグナル処理のサポート
- メッセージキューと`TCP-Socket`メッセージデリバリ機能のサポート
### UserProcess

`UserProcess` は、[addProcess](/server/methods?id=addprocess) を使用して追加されたユーザー定義の作業プロセスで、通常は特別なタスクを実行するために作成された特別な作業プロセスを作成するために使用されます。

`UserProcess` は [Managerプロセス](/learn?id=manager进程) にホストされますが、[Workerプロセス](/learn?id=worker进程) と比較して独立したプロセスであり、カスタム機能を実行するために使用されます。
