# Swoole\Process

Swooleのプロセス管理モジュールは、PHPの`pcntl`を置き換えるために提供されています。

!> このモジュールはかなり低レベルであり、オペレーティングシステムのプロセス管理をカプセル化しています。使用者は`Linux`システムでのマルチプロセスプログラミングの経験を持っている必要があります。

`PHP`に組み込まれている`pcntl`には、次のような多くの不満があります：

- プロセス間通信を提供していない
- 標準入力と出力のリダイレクトをサポートしていない
- `fork`などの原始的なインターフェースしか提供せず、誤った使用が容易

`Process`は`pcntl`よりも強力な機能と使いやすい`API`を提供し、PHPがマルチプロセスプログラミングでより簡単に行えるようにします。

`Process`が提供する機能には次のものがあります：

- プロセス間通信を簡単に実装できる
- 標準入力と出力のリダイレクトをサポートし、子プロセス内での`echo`は画面に表示されず、パイプに書き込まれ、キーボード入力はパイプからデータを読み取るためにリダイレクトできる
- [exec](/process/process?id=exec)インターフェースを提供し、作成されたプロセスは他のプログラムを実行でき、元の`PHP`親プロセスと簡単に通信できる
- コルーチン環境では`Process`モジュールを使用できないため、`runtime hook`+`proc_open`を使用して実装することができます。詳細は[Coroutine Process Management](/coroutine/proc_open)を参照してください。
```php
use Swoole\Process;

for ($n = 1; $n <= 3; $n++) {
    $process = new Process(function () use ($n) {
        echo '子进程 #' . getmypid() . " 开始并睡眠 {$n} 秒" . PHP_EOL;
        sleep($n);
        echo '子进程 #' . getmypid() . ' 退出' . PHP_EOL;
    });
    $process->start();
}
for ($n = 3; $n--;) {
    $status = Process::wait(true);
    echo "回收 #{$status['pid']}, 代码={$status['code']}, 信号={$status['signal']}" . PHP_EOL;
}
echo '主进程 #' . getmypid() . ' 退出' . PHP_EOL;
```
このドキュメントでは、属性について説明しています。

```
<name>John</name>
<age>30</age>
<gender>Male</gender>
```
### パイプ

[unixSocket](/learn?id=什么是IPC)のファイル記述子。

```php
public int $pipe;
```
### msgQueueId

メッセージキューの`id`。

```php
public int $msgQueueId;
```
### msgQueueKey

メッセージキューの`key`。

```php
public string $msgQueueKey;
```
### pid

現在のプロセスの`pid`。

```php
public int $pid;
```
### id

現在のプロセスの`id`。

```php
public int $id;
```
## 定数
パラメータ | 効果
---|---
Swoole\Process::IPC_NOWAIT | メッセージキューにデータがない場合、すぐに戻ります
Swoole\Process::PIPE_READ | 読み込みソケットを閉じる
Swoole\Process::PIPE_WRITE | 書き込みソケットを閉じる
## Methods
### __construct()

コンストラクタ。

```php
Swoole\Process->__construct(callable $function, bool $redirect_stdin_stdout = false, int $pipe_type = SOCK_DGRAM, bool $enable_coroutine = false)
```

* **パラメータ** 

  * **`callable $function`**
    * **機能**：子プロセスが正常に作成された後に実行する関数【内部では関数が自動的にオブジェクトの`callback`プロパティに保存されます。注意：このプロパティは`private`であり、クラス内部でのみアクセスできます。】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`bool $redirect_stdin_stdout`**
    * **機能**：子プロセスの標準入力と出力をリダイレクトします。【このオプションを有効にすると、子プロセス内の出力は画面に表示されず、代わりに親プロセスのパイプに書き込まれます。キーボード入力はパイプからデータを読み込むようになります。デフォルトはブロッキング読み取りです。[exec()](/process/process?id=exec)メソッドを参照してください。】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $pipe_type`**
    * **機能**：[unixSocket](/learn?id=什么是IPC) の種類【`$redirect_stdin_stdout`を有効にすると、このオプションはユーザーの引数を無視し、強制的に`SOCK_STREAM`に設定されます。子プロセス内でプロセス間通信がない場合は`0`に設定できます。】
    * **デフォルト値**：`SOCK_DGRAM`
    * **その他の値**：`0`、`SOCK_STREAM`

  * **`bool $enable_coroutine`**
    * **機能**：`callback function`内でコルーチンを有効にし、有効にすると子プロセスの関数内で直接コルーチンAPIを使用できます。
    * **デフォルト値**：`false`
    * **その他の値**：`true`
    * **バージョンの影響**：Swoole バージョン >= v4.3.0

* **[unixSocket](/learn?id=什么是IPC) の種類**

unixSocketの種類 | 説明
---|---
0 | 作成しない
1 | [SOCK_STREAM](/learn?id=什么是IPC) タイプのunixSocketを作成
2 | [SOCK_DGRAM](/learn?id=什么是IPC) タイプのunixSocketを作成
### useQueue()

プロセス間通信にメッセージキューを使用します。

```php
Swoole\Process->useQueue(int $key = 0, int $mode = SWOOLE_MSGQUEUE_BALANCE, int $capacity = -1): bool
```

* **Parameters**

  * **`int $key`**
    * **Description**: メッセージキューのキー。0以下の値を渡すと、`ftok`関数が実行中のファイル名を使用して対応するキーを生成します。
    * **Default**: `0`
    * **Other values**: なし

  * **`int $mode`**
    * **Description**: プロセス間通信モード。
    * **Default**: `SWOOLE_MSGQUEUE_BALANCE`。`Swoole\Process::pop()`はキューの最初のメッセージを返し、`Swoole\Process::push()`はメッセージに特定のタイプを追加しません。
    * **Other values**: `SWOOLE_MSGQUEUE_ORIENT`。`Swoole\Process::pop()`はキュー内のメッセージタイプが`プロセスid + 1`の特定のデータを返し、`Swoole\Process::push()`はメッセージに`プロセスid + 1`のタイプを追加します。

  * **`int $capacity`**
    * **Description**: メッセージキューに保存できるメッセージの最大数。
    * **Default**: `-1`
    * **Other values**: なし

* **Note**

  * メッセージキューにデータがない場合、`Swoole\Porcess->pop()`はブロックし続けるか、メッセージキューに新しいデータを保持する空きスペースがない場合、`Swoole\Porcess->push()`もブロックし続けます。ブロックを避けたい場合、`$mode`の値は `SWOOLE_MSGQUEUE_BALANCE|Swoole\Process::IPC_NOWAIT` または `SWOOLE_MSGQUEUE_ORIENT|Swoole\Process::IPC_NOWAIT` である必要があります。
### statQueue()

メッセージキューの状態を取得します

```php
Swoole\Process->statQueue(): array|false
```

* **Return value**

  * 配列が成功したことを表し、2つのキー値を含みます。`queue_num`は現在のキュー内のメッセージの総数を、`queue_bytes`は現在のキュー内のメッセージの総サイズを示します。
  * 失敗した場合は `false` を返します。
### freeQueue()

メッセージキューを破棄します。

```php
Swoole\Process->freeQueue(): bool
```

* **Return Value** 

  * 成功時は`true`を返します。
  * 失敗時は`false`を返します。
### pop()

メッセージキューからデータを取得します。

```php
Swoole\Process->pop(int $size = 65536): string|false
```

* **パラメーター**

  * **`int $size`**
    * **説明**：取得するデータのサイズ。
    * **デフォルト値**：`65536`
    * **その他の値**：`なし`

* **戻り値**

  * 成功した場合は`string`を返します。
  * 失敗した場合は`false`を返します。

* **注意**

  * メッセージキューのタイプが`SW_MSGQUEUE_BALANCE`の場合、キューの最初の情報を返します。
  * メッセージキューのタイプが`SW_MSGQUEUE_ORIENT`の場合、キューの最初の情報のタイプが現在の`プロセスid + 1`であるものを返します。
### push()

データをメッセージキューに送信します。

```php
Swoole\Process->push(string $data): bool
```

* **パラメータ** 

  * **`string $data`**
    * **目的**：送信するデータ。
    * **デフォルト値**：``
    * **その他の値**：`なし`


* **戻り値** 

  * 成功した場合は`true`を返します。
  * 失敗した場合は`false`を返します。

* **注意**

  * メッセージキューのタイプが`SW_MSGQUEUE_BALANCE`の場合、データはメッセージキューに直接挿入されます。
  * メッセージキューのタイプが`SW_MSGQUEUE_ORIENT`の場合、データは現在の`プロセスID + 1`を持つ型が追加されます。
### setTimeout()

设置消息队列读写超时。

```php
Swoole\Process->setTimeout(float $seconds): bool
```

* **参数** 

  * **`float $seconds`**
    * **功能**：超时时间
    * **默认值**：`无`
    * **其它值**：`无`


* **返回值** 

  * 成功返回`true`。
  * 失败返回`false`。
### setBlocking()

設定消息隊列的套接字是否阻塞。

```php
Swoole\Process->setBlocking(bool $$blocking): void
```

* **参数** 

  * **`bool $blocking`**
    * **功能**：是否阻塞，`true`表示阻塞，`false`表示不阻塞
    * **默认值**：`無`
    * **其它值**：`無`

* **注意**

  * 新创建的进程套接字默认是阻塞的，所以在做UNIX域套接字通信时，发送或读取消息会使进程阻塞。
### write()

父子プロセス間のメッセージ書き込み（UNIXドメインソケット）。

```php
Swoole\Process->write(string $data): false|int
```

* **パラメータ**

  * **`string $data`**
    * **機能**：書き込むデータ
    * **デフォルト値**：`なし`
    * **その他の値**：`なし`


* **戻り値**

  * 成功時は`int`が返され、成功した書き込まれたバイト数を表します。
  * 失敗時は`false`が返されます。
### read()

父子プロセス間のメッセージ読み取り（UNIXドメインソケット）。

```php
Swoole\Process->read(int $size = 8192): false|string
```

* **パラメーター**

  * **`int $size`**
    * **機能**：読み取るデータのサイズ
    * **デフォルト値**：`8192`
    * **その他の値**：`無し`

* **戻り値**

  * 成功した場合は`string`を返します。
  * 失敗した場合は`false`を返します。
### set()

Setting parameters.

```php
Swoole\Process->set(array $settings): void
```

You can use `enable_coroutine` to control whether to enable coroutines, which has the same effect as the fourth parameter of the constructor.

```php
Swoole\Process->set(['enable_coroutine' => true]);
```

!> Available in Swoole version >= v4.4.4
### start()

`fork`システムコールを実行して、子プロセスを起動します。`Linux`システムではプロセスを作成するのに数百マイクロ秒かかります。

```php
Swoole\Process->start(): int|false
```

* **返り値**

  * 成功時は子プロセスの`PID`
  * 失敗時は`false`が返ります。エラーコードとエラーメッセージを取得するには[swoole_errno](/functions?id=swoole_errno) および [swoole_strerror](/functions?id=swoole_strerror) を使用できます。

* **注意点**

  * 子プロセスは親プロセスのメモリとファイルハンドルを継承します
  * 子プロセスの起動時には、親プロセスから継承した[EventLoop](/learn?id=什么是eventloop)、[Signal](/process/process?id=signal)、[Timer](/timer)がクリアされます
  
  !> 子プロセスは、親プロセスのメモリとリソースを引き継ぐため、親プロセス内でRedis接続が作成されている場合、子プロセス内でもこのオブジェクトが保持され、すべての操作が同じ接続に対して行われます。以下に例を示します。

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);

function callback_function() {
    swoole_timer_after(1000, function () {
        echo "hello world\n";
    });
    global $redis;//同じ接続
};

swoole_timer_tick(1000, function () {
    echo "parent timer\n";
});//継承されない

Swoole\Process::signal(SIGCHLD, function ($sig) {
    while ($ret = Swoole\Process::wait(false)) {
        // 新しい子プロセスを作成
        $p = new Swoole\Process('callback_function');
        $p->start();
    }
});

// 新しい子プロセスを作成
$p = new Swoole\Process('callback_function');

$p->start();
```

!> 1. 子プロセスが起動した後、親プロセスで[Swoole\Timer::tick](/timer?id=tick)で作成されたタイマーや、[Process::signal](/process/process?id=signal)で監視されている信号、[Swoole\Event::add](/event?id=add)で追加されたイベントリスナーが自動的にクリアされます。  
2. 子プロセスは、親プロセスが作成した`$redis`接続オブジェクトを継承し、親子プロセスが同じ接続を使用します。
### exportSocket()

`unixSocket`を`Swoole\Coroutine\Socket`オブジェクトとしてエクスポートし、`Swoole\Coroutine\socket`オブジェクトのメソッドを使用してプロセス間通信を行います。具体的な使用法については、[Coroutine\socket](/coroutine_client/socket)と[IPC通信](/learn?id=什么是IPC)を参照してください。

```php
Swoole\Process->exportSocket(): Swoole\Coroutine\Socket|false
```

!> このメソッドを複数回呼び出しても、返されるオブジェクトは同じものになります。  
`exportSocket()`でエクスポートされた`socket`は新しい`fd`であり、エクスポートされた`socket`を閉じても元のプロセスのパイプに影響を与えません。  
`Swoole\Coroutine\Socket`オブジェクトであるため、[協力コンテナ](/coroutine/scheduler)内で使用する必要があり、Swoole\Processのコンストラクタの`$enable_coroutine`パラメータはtrueである必要があります。  
同様に親プロセスが`Swoole\Coroutine\Socket`オブジェクトを使用する場合は、手動で`Coroutine\run()`を使用して協力コンテナを作成する必要があります。

* **戻り値**

  * 成功した場合は`Coroutine\Socket`オブジェクトが返されます
  * プロセスがunixSocketを作成していない場合は操作が失敗し、`false`が返されます

* **使用例**

簡単な親子プロセス間通信を実現しました：

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    echo $socket->recv();
    $socket->send("hello master\n");
    echo "proc1 stop\n";
}, false, 1, true);

$proc1->start();

// 親プロセスは協力コンテナを作成
run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    $socket->send("hello pro1\n");
    var_dump($socket->recv());
});
Process::wait(true);
```

より複雑な通信の例：

```php
use Swoole\Process;
use Swoole\Timer;
use function Swoole\Coroutine\run;

$process = new Process(function ($proc) {
    Timer::tick(1000, function () use ($proc) {
        $socket = $proc->exportSocket();
        $socket->send("hello master\n");
        echo "child timer\n";
    });
}, false, 1, true);

$process->start();

run(function() use ($process) {
    Process::signal(SIGCHLD, static function ($sig) {
        while ($ret = Swoole\Process::wait(false)) {
            /* クリーンアップを行い、イベントループが終了します */
            Process::signal(SIGCHLD, null);
            Timer::clearAll();
        }
    });
    /* ここで他の非同期コードや協力コードを実行できます */
    Timer::tick(500, function () {
        echo "parent timer\n";
    });

    $socket = $process->exportSocket();
    while (1) {
        var_dump($socket->recv());
    }
});
```
!> デフォルトのタイプは`SOCK_STREAM`であり、TCPデータパケットの境界問題を処理する必要があります。[Coroutine\socket](/coroutine_client/socket)の`setProtocol()`メソッドを参照してください。

`SOCK_DGRAM`タイプを使用してIPC通信を行うと、TCPデータパケットの境界問題を回避できます。[IPC通信](/learn?id=什么是IPC)を参照してください：

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

// SOCK_DGRAM タイプのソケットでも、sendto / recvfrom の一連の関数を使用する必要はありません。send/recv を使用してください。
$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    while (1) {
        var_dump($socket->send("hello master\n"));
    }
    echo "proc1 stop\n";
}, false, 2, 1);//コンストラクタのpipe typeを2に設定するとSOCK_DGRAMになります

$proc1->start();

run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    Swoole\Coroutine::sleep(5);
    var_dump(strlen($socket->recv()));//1回のrecvで1つの"hello master\n"文字列しか受信されないことに注意してください
});

Process::wait(true);
```  
### name()

Process name change function. This function is an alias for [swoole_set_process_name](/functions?id=swoole_set_process_name).

```php
Swoole\Process->name(string $name): bool
```

!> After executing `exec`, the process name will be reset by the new program; the `name` method should be used in the callback function of the child process after `start`.
### exec()

`exec()`関数は外部プログラムを実行し、これは`exec`システムコールのラッパーです。

```php
Swoole\Process->exec(string $execfile, array $args);
```

* **パラメータ** 

  * **`string $execfile`**
    * **動作**：実行ファイルの絶対パスを指定します。例：`"/usr/bin/python"`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`array $args`**
    * **動作**：`exec`の引数リスト【例：`array('test.py', 123)`は`python test.py 123`と同義です】
    * **デフォルト値**：なし
    * **その他の値**：なし

成功すると、現在のプロセスのコードセグメントが新しいプログラムで置き換えられます。子プロセスは新しいプログラムに変換されます。親プロセスと現在のプロセスはまだ親子関係にあります。

親プロセスと新プロセスは、標準入出力を介して通信することができますが、標準入出力のリダイレクトを有効にする必要があります。

!> `$execfile`は絶対パスを使用する必要があります。そうでない場合、ファイルが存在しないエラーが発生する可能性があります。  
`exec`システムコールは指定されたプログラムで現在のプログラムを上書きするため、子プロセスは親プロセスと標準出力を読み書きして通信する必要があります。  
`redirect_stdin_stdout = true`が指定されていない場合、`exec`を実行した後に子プロセスと親プロセスが通信できなくなります。

* **使用例**

例1： `Swoole\Process`で作成された子プロセスで[Swoole\Server](/server/init)を使用できますが、安全のために`$process->start`でプロセスを作成した後に`$worker->exec()`を呼び出す必要があります。コード例：

```php
$process = new Swoole\Process('callback_function', true);

$pid = $process->start();

function callback_function(Swoole\Process $worker)
{
    $worker->exec('/usr/local/bin/php', array(__DIR__.'/swoole_server.php'));
}

Swoole\Process::wait();
```

例2：Yiiプログラムの起動

```php
$process = new \Swoole\Process(function (\Swoole\Process $childProcess) {
    // この書き方はサポートされていません
    // $childProcess->exec('/usr/local/bin/php /var/www/project/yii-best-practice/cli/yii t/index -m=123 abc xyz');

    // execシステムコールをカプセル化する
    // 絶対パス
    // パラメータは配列に分ける必要があります
    $childProcess->exec('/usr/local/bin/php', ['/var/www/project/yii-best-practice/cli/yii', 't/index', '-m=123', 'abc', 'xyz']); // execシステムコール
});
$process->start(); // 子プロセスを起動
```

例3：親プロセスと`exec`子プロセスは標準入出力を使用して通信します：

```php
// exec - execプロセスとの間でパイプ通信を行う
use Swoole\Process;
use function Swoole\Coroutine\run;

$process = new Process(function (Process $worker) {
    $worker->exec('/bin/echo', ['hello']);
}, true, 1, true); // 標準入出力のリダイレクトを有効にする必要があります

$process->start();

run(function() use($process) {
    $socket = $process->exportSocket();
    echo "from exec: " . $socket->recv() . "\n";
});
```

例4：シェルコマンドの実行

`exec`メソッドは`PHP`の`shell_exec`とは異なり、より低レベルなシステムコールのラッパーです。シェルコマンドを実行する場合は以下の方法を使用してください：

```php
$worker->exec('/bin/sh', array('-c', "cp -rf /data/test/* /tmp/test/"));
```
```php
Swoole\Process->close(int $which): bool
```

* **Parameters**

  * **`int $which`**
    * **Function**: Since a unixSocket is full duplex, specify which end to close [default is `0` to close both read and write, `1`: to close write, `2` to close read].
    * **Default**: `0`, closing the read and write socket.
    * **Other values**: `Swoole/Process::SW_PIPE_CLOSE_READ` close the read socket, `Swoole/Process::SW_PIPE_CLOSE_WRITE` close the write socket.

!> In some special cases, `Process` objects cannot be released, and continuously creating processes will lead to connection leaks. Call this function to directly close the `unixSocket` and free up resources.``` 
```php
Swoole\Process->exit(int $status = 0);
```

* **パラメーター** 

  * **`int $status`**
    * **機能**：プロセスの終了ステータスコード【`0`の場合は正常終了し、クリーンアップ作業を継続します】
    * **デフォルト値**：`0`
    * **その他の値**：なし

!> クリーンアップ作業には次のものが含まれます：

  * `PHP`の`shutdown_function`
  * オブジェクトのデストラクター（`__destruct`）
  * 他の拡張機能の`RSHUTDOWN`関数

`$status`が`0`でない場合、異常終了を意味し、プロセスは即座に終了し、関連するクリーンアップ作業は実行されません。

親プロセスでは、`Process::wait`を実行することで子プロセスの終了イベントとステータスコードを取得できます。
### kill()

指定の`pid`プロセスにシグナルを送信します。

```php
Swoole\Process::kill(int $pid, int $signo = SIGTERM): bool
```

* **パラメータ** 

  * **`int $pid`**
    * **意味**：プロセス`pid`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $signo`**
    * **意味**：送信されるシグナル【`$signo=0`は、プロセスの存在をチェックできますが、シグナルは送信されません】
    * **デフォルト値**：`SIGTERM`
    * **その他の値**：なし
### signal()

非同期シグナルリスナーを設定します。

```php
Swoole\Process::signal(int $signo, callable $callback): bool
```

このメソッドは`signalfd`および[EventLoop](/learn?id=什么是eventloop)に基づいて非同期`IO`を実行し、ブロッキングプログラムでは使用できません。これにより、登録されたリスナーコールバック関数がスケジュールされなくなります。

同期ブロッキングプログラムでは、`pcntl`拡張から提供される`pcntl_signal`を使用できます。

このシグナルのコールバック関数がすでに設定されている場合、再設定すると過去の設定が上書きされます。

* **パラメーター**

  * **`int $signo`**
    * **説明**：シグナル
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $callback`**
    * **説明**：コールバック関数【`$callback`が`null`の場合、シグナルリスニングが解除されます】
    * **デフォルト値**：なし
    * **その他の値**：なし

!> [Swoole\Server](/server/init)内では、`SIGTERM`や`SIGALAM`など、特定のシグナルリスナーを設定できません

* **使用例**

```php
Swoole\Process::signal(SIGTERM, function($signo) {
     echo "shutdown.";
});
```

!> `v4.4.0`で、プロセスの[EventLoop](/learn?id=什么是eventloop)にシグナルリスニングのイベントしかなく、他のイベント（例：タイマーなど）がない場合、プロセスは直ちに終了します。

```php
Swoole\Process::signal(SIGTERM, function($signo) {
     echo "shutdown.";
});
Swoole\Event::wait();
```

前述のプログラムは[EventLoop](/learn?id=什么是eventloop)に入りませんが、`Swoole\Event::wait()`は直ちに返り、プロセスが終了します。
### wait()

終了した子プロセスを回収します。

!> Swoole のバージョンが `v4.5.0` 以上の場合、`wait()` のコルーチンバージョンを使用することが推奨されます。詳細は[Swoole\Coroutine\System::wait()](/coroutine/system?id=wait)を参照してください。

```php
Swoole\Process::wait(bool $blocking = true): array|false
```

* **パラメータ**

  * **`bool $blocking`**
    * **機能**：ブロッキング待機するかどうかを指定します【デフォルトはブロッキング】
    * **デフォルト値**：`true`
    * **その他の値**：`false`

* **戻り値**

  * 成功時：子プロセスの`PID`、終了ステータスコード、KILL されたシグナルが含まれる配列が返されます。
  * 失敗時：`false` が返されます。

!> 各子プロセスが終了するごとに、親プロセスは必ず `wait()` を実行して回収する必要があります。それ以外の場合、子プロセスはゾンビプロセスになり、オペレーティングシステムのプロセスリソースが無駄になります。  
親プロセスに他のタスクがあって `wait` でブロックすることができない場合、親プロセスは終了したプロセスに対して `SIGCHLD` シグナルを登録して `wait` を実行する必要があります。  
SIGCHILD シグナルが発生すると、複数の子プロセスが同時に終了する可能性があります。`wait()` を非ブロッキングモードに設定し、`false` が返されるまで `wait` をループして実行する必要があります。

* **例**

```php
Swoole\Process::signal(SIGCHLD, function ($sig) {
    // ブロッキングモードであってはいけない（false 必須）
    while ($ret = Swoole\Process::wait(false)) {
        echo "PID={$ret['pid']}\n";
    }
});
```
### daemon()

現在のプロセスをデーモンプロセスに変身させます。

```php
Swoole\Process::daemon(bool $nochdir = true, bool $noclose = true): bool
```

* **パラメータ** 

  * **`bool $nochdir`**
    * **目的**：現在のディレクトリをルートディレクトリに変更するかどうか【`true`の場合は現在のディレクトリをルートディレクトリに変更しない】
    * **デフォルト値**：`true`
    * **他の値**：`false`

  * **`bool $noclose`**
    * **目的**：標準入力と標準出力のファイルディスクリプタを閉じるかどうか【`true`の場合は標準入力と標準出力のファイルディスクリプタを閉じない】
    * **デフォルト値**：`true`
    * **他の値**：`false`

!> デーモンプロセスに変身すると、そのプロセスの`PID`が変化します。現在の`PID`を取得するには`getmypid()`を使用できます。
### alarm()

`alarm`関数は、高精度タイマーであり、オペレーティングシステムの`setitimer`システムコールのラッパーであり、マイクロ秒単位のタイマーを設定できます。タイマーはシグナルをトリガーし、[Process::signal](/process/process?id=signal)または`pcntl_signal`と組み合わせて使用する必要があります。

!> `alarm`関数は[Timer](/timer)と同時に使用できません

```php
Swoole\Process->alarm(int $time, int $type = 0): bool
```

* **Parameters** 

  * **`int $time`**
    * **Description**：タイマーのインターバル時間【負数の場合、タイマーをクリアします】
    * **Value Unit**：マイクロ秒
    * **Default Value**：なし
    * **Other Values**：なし
  
  * **`int $type`**
    * **Description**：タイマーのタイプ
    * **Default Value**：`0`
    * **Other Values**：

タイマーのタイプ | 説明
---|---
0 | 実時間を表し、`SIGALAM`シグナルをトリガーします
1 | ユーザーモードCPU時間を表し、`SIGVTALAM`シグナルをトリガーします
2 | ユーザーモード+カーネルモード時間を表し、`SIGPROF`シグナルをトリガーします

* **Return Value**

  * 成功時は`true`を返します
  * 失敗時は`false`を返し、`swoole_errno`を使用してエラーコードを取得できます

* **Example**

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

run(function () {
    Process::signal(SIGALRM, function () {
        static $i = 0;
        echo "#{$i}\talarm\n";
        $i++;
        if ($i > 20) {
            Process::alarm(-1);
            Process::kill(getmypid());
        }
    });

    //100ms
    Process::alarm(100 * 1000);

    while(true) {
        sleep(0.5);
    }
});
```
### setAffinity()

`CPU`アフィニティを設定すると、プロセスを特定の`CPU`コアにバインドできます。

この関数の目的は、プロセスを特定の`CPU`コアで実行させ、一部の`CPU`リソースを優先度の高いプログラムに割り当てることです。

```php
Swoole\Process->setAffinity(array $cpus): bool
```

* **パラメータ** 

  * **`array $cpus`**
    * **機能**：`CPU`コアにバインドする 【例： `array(0,2,3)` は `CPU0/CPU2/CPU3` にバインド】
    * **デフォルト値**：なし
    * **その他の値**：なし

!> -`$cpus`内の要素数は`CPU`コア数を超えてはいけません；  
-`CPU-ID`は（`CPU`コア数 - `1`）を超えてはいけません；  
-この関数はオペレーティングシステムの`CPU`バインド機能をサポートする必要があります；  
-[swoole_cpu_num()](/functions?id=swoole_cpu_num) 関数を使用して、現在のサーバーの`CPU`コア数を取得できます。
### setPriority()

プロセス、プロセスグループ、およびユーザープロセスの優先度を設定します。

!> Swooleバージョン >= `v4.5.9` で利用可能

```php
Swoole\Process->setPriority(int $which, int $priority): bool
```

* **引数**

  * **`int $which`**
    * **説明**：優先度を変更する種類を決定します
    * **デフォルト値**：なし
    * **その他の値**：

| 定数         | 説明       |
| ------------ | ---------- |
| PRIO_PROCESS | プロセス   |
| PRIO_PGRP    | プロセスグループ |
| PRIO_USER    | ユーザープロセス |

  * **`int $priority`**
    * **説明**：優先度。値が小さいほど優先度が高いです
    * **デフォルト値**：なし
    * **その他の値**：`[-20, 20]`

* **戻り値**

  * `false`を返す場合、[swoole_errno](/functions?id=swoole_errno)および[swoole_strerror](/functions?id=swoole_strerror)を使用してエラーコードとエラーメッセージを取得できます。
### getPriority()

プロセスの優先度を取得します。

!> Swooleバージョン >= `v4.5.9` で使用可能

```php
Swoole\Process->getPriority(int $which): int
```
