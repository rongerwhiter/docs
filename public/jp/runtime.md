# ランタイム

`Swoole4+`は`Swoole1.x`に比べて、協力的なコルーチンを提供し、すべてのビジネスコードは同期的ですが、バックグラウンドのIOは非同期です。 これにより、伝統的な非同期コールバックに伴う分散したコードロジックや多重コールバックに陥ることでコードの保守性が損なわれることが避けられ、同時に並行性が確保されます。この効果を達成するためには、すべての`IO`リクエストが[非同期IO](/learn?id=同期io非同期io)である必要があります。`Swoole1.x`時代に提供されていた`MySQL`、`Redis`などのクライアントは非同期IOでしたが、コルーチン方式ではなく非同期コールバック方式であったため、`Swoole4`時代にこれらのクライアントを削除しました。

これらのクライアントのコルーチンサポート問題を解決するために、Swoole開発チームは多くの作業を行いました：

- 最初は、各種類のクライアントに対してコルーチンクライアントを作成しましたが、次の３つの問題がありました：

  * 実装が複雑で、すべてのクライアントの微細なプロトコルを完璧にサポートするのは非常に手間がかかります。
  * ユーザーが変更する必要のあるコードがかなり多くあります。たとえば、以前は`MySQL`のクエリをPHPのネイティブ`PDO`で行っていましたが、今では[Swoole\Coroutine\MySQL](/coroutine_client/mysql)のメソッドを使用する必要があります。
  * すべての操作を網羅するのが難しい場合があります。たとえば、`proc_open()`、`sleep()`関数などがブロックし、プログラムが同期ブロッキングになる可能性があります。

- 上記の問題に対処するために、Swoole開発チームは実装アプローチを変更し、コルーチンクライアントを実現するために、`Hook`ネイティブPHP関数の方法を採用しました。 これにより、1行のコードで以前の同期IOコードをコルーチンスケジューリングできる[非同期IO](/learn?id=同期io非同期io)に変換できるようになり、つまり`ワンクリックでコルーチン化`が可能です。

!> この機能は`v4.3`以降で安定化し、コルーチン化できる関数がますます増えています。したがって、以前に書かれた一部のコルーチンクライアントはもはや推奨されていません。 詳細については、[コルーチンクライアント](/coroutine_client/init)を参照してください。 たとえば、`v4.3+`ではファイル操作（`file_get_contents`、`fread`など）の`コルーチン化`がサポートされており、`v4.3+`バージョンを使用している場合は、Swooleが提供する[コルーチンファイル操作](/coroutine/system)を使用せずに`コルーチン化`を直接使用できます。
## 関数のプロトタイプ

`flags`を使って`Coroutine`化する関数の範囲を設定します。

```php
Co::set(['hook_flags'=> SWOOLE_HOOK_ALL]); // v4.4+のバージョンで使用します。
// または
Swoole\Runtime::enableCoroutine($flags = SWOOLE_HOOK_ALL);
```

複数の`flags`を同時に有効にする場合は`|`演算子を使用する必要があります。

```php
Co::set(['hook_flags'=> SWOOLE_HOOK_TCP | SWOOLE_HOOK_SLEEP]);
```

!> `Hook`された関数は[Coroutineコンテナ](/coroutine/scheduler)で使用する必要があります。
#### よくある質問 :id=runtime-qa

!> **`Swoole\Runtime::enableCoroutine()`と`Co::set(['hook_flags'])`、どちらを使えばいいですか**

* `Swoole\Runtime::enableCoroutine()`はサービス起動後（実行時）、フラグを動的に設定でき、このメソッドを呼び出した後、現在のプロセス全体に適用されます。プロジェクト全体でカバレッジを100%網羅するためには、プロジェクト全体開始時に配置する必要があります。
* `Co::set()`はPHPの`ini_set()`のように考えることができ、[Server->start()](/server/methods?id=start)の前、または[Co\run()](/coroutine/scheduler)の前に呼び出す必要があります。それ以外の場合、設定された`hook_flags`は有効になりません。`v4.4+`のバージョンでは、この方法で`flags`を設定する必要があります。
* `Co::set(['hook_flags'])`や`Swoole\Runtime::enableCoroutine()`のどちらを使っても、1度だけ呼び出すべきです。繰り返し呼び出すと上書きされます。
```python
['-a', '--all', '显示所有标志']
['-f', '--file', '指定文件路径']
['-v', '--verbose', '显示详细信息']
['-h', '--help', '显示帮助信息']
```  
### SWOOLE_HOOK_ALL

以下のすべての種類のフラグを開きます（CURLを除く）

!> v4.5.4以降、`SWOOLE_HOOK_ALL` には `SWOOLE_HOOK_CURL` が含まれています。

```php
Co::set(['hook_flags' => SWOOLE_HOOK_ALL]); //CURLを除く
Co::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_CURL]); //CURLを含むすべての種類を本当のコルーチン化します
```
### SWOOLE_HOOK_TCP

`v4.1`からサポートされ、TCPソケットタイプのstream、一般的な`Redis`、`PDO`、`Mysqli`、そしてPHPの [streams](https://www.php.net/streams) シリーズ関数を使用してTCP接続を操作する場合、`Hook` が可能です。以下は例です：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TCP]);

Co\run(function() {
    for ($c = 100; $c--;) {
        go(function () {//100個のコルーチンを作成
            $redis = new Redis();
            $redis->connect('127.0.0.1', 6379);//ここでコルーチンスケジューリングが発生し、CPUは次のコルーチンに移動し、プロセスをブロックしません
            $redis->get('key');//ここでコルーチンスケジューリングが発生し、CPUは次のコルーチンに移動し、プロセスをブロックしません
        });
    }
});
```

上記のコードでは、元の`Redis`クラスが使用されていますが、実際には`非同期IO`になっています。`Co\run()` はコルーチンコンテナを作成し、`go()` はコルーチンを作成することで、これらの操作は[Swoole\Serverクラス群](/server/init)で自動的に処理されます。手動で行う必要はありません。[enable_coroutine](/server/setting?id=enable_coroutine) を参照してください。

つまり、従来の`PHP`プログラマーは、最も馴染みのあるロジックコードを使用して、高並行性で高性能なプログラムを書けるようになりました。以下は例です：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TCP]);

$http = new Swoole\Http\Server("0.0.0.0", 9501);
$http->set(['enable_coroutine' => true]);

$http->on('request', function ($request, $response) {
      $redis = new Redis();
      $redis->connect('127.0.0.1', 6379);//ここでコルーチンスケジューリングが発生し、CPUは次のコルーチン(次のリクエスト)に移動し、プロセスをブロックしません
      $redis->get('key');//ここでコルーチンスケジューリングが発生し、CPUは次のコルーチン(次のリクエスト)に移動し、プロセスをブロックしません
});

$http->start();
```
### SWOOLE_HOOK_UNIX

`v4.2`からサポートされています。`Unix Stream Socket`タイプのstreamの例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UNIX]);

Co\run(function () {
    $socket = stream_socket_server(
        'unix://swoole.sock',
        $errno,
        $errstr,
        STREAM_SERVER_BIND | STREAM_SERVER_LISTEN
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_accept($socket)) {
    }
});
```
### SWOOLE_HOOK_UDP

`v4.2`からサポートされています。UDPソケットタイプのストリーム、例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UDP]);

Co\run(function () {
    $socket = stream_socket_server(
        'udp://0.0.0.0:6666',
        $errno,
        $errstr,
        STREAM_SERVER_BIND
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_recvfrom($socket, 1, 0)) {
    }
});
```
### SWOOLE_HOOK_UDG

`v4.2`からサポートされています。Unix Dgram Socket型のストリームの例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UDG]);

Co\run(function () {
    $socket = stream_socket_server(
        'udg://swoole.sock',
        $errno,
        $errstr,
        STREAM_SERVER_BIND
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_recvfrom($socket, 1, 0)) {
    }
});
```
### SWOOLE_HOOK_SSL

`v4.2`からサポートされています。SSLソケットタイプのストリーム、例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SSL]);

Co\run(function () {
    $host = 'host.domain.tld';
    $port = 1234;
    $timeout = 10;
    $cert = '/path/to/your/certchain/certchain.pem';
    $context = stream_context_create(
        array(
            'ssl' => array(
                'local_cert' => $cert,
            )
        )
    );
    if ($fp = stream_socket_client(
        'ssl://' . $host . ':' . $port,
        $errno,
        $errstr,
        30,
        STREAM_CLIENT_CONNECT,
        $context
    )) {
        echo "connected\n";
    } else {
        echo "ERROR: $errno - $errstr \n";
    }
});
```  
### SWOOLE_HOOK_TLS

`v4.2` からサポートされています。TLS ソケットタイプの stream 、[参考](https://www.php.net/manual/en/context.ssl.php)。

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TLS]);
```
### SWOOLE_HOOK_SLEEP

`v4.2`からサポートされています。`sleep`関数の`Hook`は`sleep`、`usleep`、`time_nanosleep`、`time_sleep_until`を含みます。内部タイマーの最小単位が`1ms`なので、`usleep`などの高精度なスリープ関数を使用すると、`1ms`未満に設定した場合は直接`sleep`システムコールが使用されます。非常に短いスリープブロッキングが発生する可能性があります。例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SLEEP]);

Co\run(function () {
    go(function () {
        sleep(1);
        echo '1' . PHP_EOL;
    });
    go(function () {
        echo '2' . PHP_EOL;
    });
});
//出力
2
1
```
### SWOOLE_HOOK_FILE

`v4.3`からサポートされています。

* **ファイル操作の`コルーチン処理`、サポートされる関数は以下です：**

    * `fopen`
    * `fread`/`fgets`
    * `fwrite`/`fputs`
    * `file_get_contents`、`file_put_contents`
    * `unlink`
    * `mkdir`
    * `rmdir`

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_FILE]);

Co\run(function () {
    $fp = fopen("test.log", "a+");
    fwrite($fp, str_repeat('A', 2048));
    fwrite($fp, str_repeat('B', 2048));
});
```
### SWOOLE_HOOK_STREAM_FUNCTION

`v4.4`からサポートされています。`stream_select()`の`Hook`の例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_STREAM_FUNCTION]);

Co\run(function () {
    $fp1 = stream_socket_client("tcp://www.baidu.com:80", $errno, $errstr, 30);
    $fp2 = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
    if (!$fp1) {
        echo "$errstr ($errno) \n";
    } else {
        fwrite($fp1, "GET / HTTP/1.0\r\nHost: www.baidu.com\r\nUser-Agent: curl/7.58.0\r\nAccept: */*\r\n\r\n");
        $r_array = [$fp1, $fp2];
        $w_array = $e_array = null;
        $n = stream_select($r_array, $w_array, $e_array, 10);
        $html = '';
        while (!feof($fp1)) {
            $html .= fgets($fp1, 1024);
        }
        fclose($fp1);
    }
});
```
### SWOOLE_HOOK_BLOCKING_FUNCTION

`v4.4`からサポートされています。ここで言う`blocking function`には、`gethostbyname`、`exec`、`shell_exec`が含まれます。例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_BLOCKING_FUNCTION]);

Co\run(function () {
    echo shell_exec('ls');
});
```
### SWOOLE_HOOK_PROC

`v4.4`からサポートされています。 `proc*` 関数をコルーチン化します。これには、 `proc_open`、 `proc_close`、 `proc_get_status`、 `proc_terminate` が含まれます。

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PROC]);

Co\run(function () {
    $descriptorspec = array(
        0 => array("pipe", "r"),  // stdin, child process read from it
        1 => array("pipe", "w"),  // stdout, child process write to it
    );
    $process = proc_open('php', $descriptorspec, $pipes);
    if (is_resource($process)) {
        fwrite($pipes[0], '<?php echo "I am process\n" ?>');
        fclose($pipes[0]);

        while (true) {
            echo fread($pipes[1], 1024);
        }

        fclose($pipes[1]);
        $return_value = proc_close($process);
        echo "command returned $return_value" . PHP_EOL;
    }
});
```
### SWOOLE_HOOK_CURL

[v4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x)後、または`v4.5`以降で正式にサポートされています。

* **CURLのHOOK、サポートされている関数：**

     * curl_init
     * curl_setopt
     * curl_exec
     * curl_multi_getcontent
     * curl_setopt_array
     * curl_error
     * curl_getinfo
     * curl_errno
     * curl_close
     * curl_reset

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_CURL]);

Co\run(function () {
    $ch = curl_init();  
    curl_setopt($ch, CURLOPT_URL, "http://www.xinhuanet.com/");  
    curl_setopt($ch, CURLOPT_HEADER, false);  
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $result = curl_exec($ch);  
    curl_close($ch);
    var_dump($result);
});
```
### SWOOLE_HOOK_NATIVE_CURL

`Coroutines processing` for native CURL.

!> Available in Swoole version >= `v4.6.0`

!> Before using, you need to enable the [--enable-swoole-curl](/environment?id=common-parameters) option at compile time;  
When this option is enabled, `SWOOLE_HOOK_NATIVE_CURL` will be automatically set, and `SWOOLE_HOOK_CURL` will be disabled;  
At the same time, `SWOOLE_HOOK_ALL` includes `SWOOLE_HOOK_NATIVE_CURL`

```php
Co::set(['hook_flags' => SWOOLE_HOOK_NATIVE_CURL]);

Co::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_NATIVE_CURL]);
```

Example:

```php
Co::set(['hook_flags' => SWOOLE_HOOK_ALL]);

Co\run(function () {
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, "http://httpbin.org/get");
    curl_setopt($ch, CURLOPT_HEADER, false);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $result = curl_exec($ch);
    curl_close($ch);
    var_dump($result);
});
```
### SWOOLE_HOOK_SOCKETS

`sockets`拡張機能に対する`コルーチン化処理`。

!> Swooleバージョン >= `v4.6.0` で利用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SOCKETS]);
```
### SWOOLE_HOOK_STDIO

`STDIO`の`コルーチン化処理`。

!> Swooleバージョン >= `v4.6.2` で利用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_STDIO]);
```

例：

```php
use Swoole\Process;
Co::set(['socket_read_timeout' => -1, 'hook_flags' => SWOOLE_HOOK_STDIO]);
$proc = new Process(function ($p) {
    Co\run(function () use($p) {
        $p->write('start'.PHP_EOL);
        go(function() {
            co::sleep(0.05);
            echo "sleep\n";
        });
        echo fread(STDIN, 1024);
    });
}, true, SOCK_STREAM);
$proc->start();
echo $proc->read();
usleep(100000);
$proc->write('hello world'.PHP_EOL);
echo $proc->read();
echo $proc->read();
Process::wait();
```
### SWOOLE_HOOK_PDO_PGSQL

`pdo_pgsql` の`コルーチン対応`。

!> Swooleのバージョン >= `v5.1.0` で利用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_PGSQL]);
```

例：
```php
<?php
function test()
{
    $dbname   = "test";
    $username = "test";
    $password = "test";
    try {
        $dbh = new PDO("pgsql:dbname=$dbname;host=127.0.0.1:5432", $username, $password);
        $dbh->exec('create table test (id int)');
        $dbh->exec('insert into test values(1)');
        $dbh->exec('insert into test values(2)');
        $res = $dbh->query("select * from test");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['trace_flags' => SWOOLE_HOOK_PDO_PGSQL]);

Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_ODBC

`pdo_odbc`の`コルーチン化処理`。

!> Swooleバージョン >= `v5.1.0` で利用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ODBC]);
```

例：
```php
<?php
function test()
{
    $username = "test";
    $password = "test";
    try {
        $dbh = new PDO("odbc:mysql-test");
        $res = $dbh->query("select sleep(1) s");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['trace_flags' => SWOOLE_TRACE_CO_ODBC, 'log_level' => SWOOLE_LOG_DEBUG]);

Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_ORACLE

`pdo_oci`の`コルーチン処理`に対応します。

!> Swooleバージョン >= `v5.1.0`で利用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ORACLE]);
```

例：
```php
<?php
function test()
{
	$tsn = 'oci:dbname=127.0.0.1:1521/xe;charset=AL32UTF8';
	$username = "test";
	$password = "test";
    try {
        $dbh = new PDO($tsn, $username, $password);
        $dbh->exec('create table test (id int)');
        $dbh->exec('insert into test values(1)');
        $dbh->exec('insert into test values(2)');
        $res = $dbh->query("select * from test");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ORACLE]);
Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_SQLITE
`pdo_sqlite`の`コルーチン処理`。

!> Swooleバージョン >= `v5.1.0` で使用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_SQLITE]);
```

* **注意**

!> `swoole`は`sqlite`データベースのコルーチン化において、[スレッドセーフ](https://www.sqlite.org/threadsafe.html)を保証するために`シリアライズモード`を採用しています。  
もし`sqlite`データベースが単一スレッドモードでコンパイルされている場合、`swoole`は`sqlite`をコルーチン化できず、警告が発生しますが、使用には影響しません。単に、CRUD操作中にコルーチンの切り替わりは発生しません。この場合、`sqlite`を再コンパイルしてスレッドモードを`シリアライズ化`または`マルチスレッド`に指定する必要があります。[理由](https://www.sqlite.org/compile.html#threadsafe)。  
コルーチン環境で作成された`sqlite`接続はすべて`シリアライズ化`されており、非コルーチン環境で作成された`sqlite`接続はデフォルトで`sqlite`のスレッドモードに従います。  
もし`sqlite`のスレッドモードが`マルチスレッド`である場合、非コルーチン環境で作成された接続は複数のコルーチンと共有されません。なぜなら、この時点でデータベース接続は`マルチスレッドモード`であり、コルーチン化環境で使用しても`シリアライズ化`されません。  
`sqlite`のデフォルトスレッドモードは`シリアライズ化`です。[シリアライズ化の説明](https://www.sqlite.org/c3ref/c_config_covering_index_scan.html#sqliteconfigserialized)，[デフォルトスレッドモード](https://www.sqlite.org/compile.html#threadsafe)。

例：
```php
<?php
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\go;

Co::set(['hook_flags'=> SWOOLE_HOOK_PDO_SQLITE]);

run(function() {
    for($i = 0; $i <= 5; $i++) {
        go(function() use ($i) {
            $db = new PDO('sqlite::memory:');
            $db->query('select randomblob(99999999)');
            var_dump($i);
        });
    }
});
```
```python
def greet():
    return "Hello!"

print(greet())
```

以下は、プログラムの出力です。
### setHookFlags()

`flags`を使用して`Hook`する関数の範囲を設定します

!> Swooleバージョン >= `v4.5.0` で利用可能

```php
Swoole\Runtime::setHookFlags(int $flags): bool
```
### getHookFlags()

現在の`Hook`コンテンツの`flags`を取得します。 これは、`Hook`を有効にする際に指定した`flags`とは異なるかもしれません（`Hook`に成功しなかった`flags`はクリアされる可能性があるため）

!> Swoole version >= `v4.4.12` is available

```php
Swoole\Runtime::getHookFlags(): int
```
## 一般的なフックリスト
### 使用可能なリスト

- `redis`拡張機能
- `pdo_mysql`、`mysqli`拡張機能（`mysqlnd`モードを使用する場合、コルーチンはサポートされません）
- `soap`拡張機能
- `file_get_contents`、`fopen`
- `stream_socket_client` (`predis`、`php-amqplib`)
- `stream_socket_server`
- `stream_select`（バージョン`4.3.2`以上が必要）
- `fsockopen`
- `proc_open`（バージョン`4.4.0`以上が必要）
- `curl`
### 利用不可

!> **コルーチン化サポートなし**は、コルーチンをブロッキングモードに変換し、そのためコルーチンを使用するメリットがなくなることを意味します

  * `mysql`：`libmysqlclient`を使用している
  * `mongo`：`mongo-c-client`を使用している
  * `pdo_pgsql`，Swooleバージョン >= `v5.1.0`では、`pdo_pgsql`がコルーチン化される
  * `pdo_oci`，Swooleバージョン >= `v5.1.0`では、`pdo_oci`がコルーチン化される
  * `pdo_odbc`，Swooleバージョン >= `v5.1.0`では、`pdo_odbc`がコルーチン化される
  * `pdo_firebird`
  * `php-amqp`
## API変更

`v4.3`およびそれ以前のバージョンでは、`enableCoroutine`のAPIには2つのパラメータが必要です。

```php
Swoole\Runtime::enableCoroutine(bool $enable = true, int $flags = SWOOLE_HOOK_ALL);
```

- `$enable`：コルーチンを有効または無効にします。
- `$flags`：コルーチン化を選択するタイプを選択します。複数選択可能で、デフォルトはすべて選択です。`$enable = true`の場合のみ有効です。

!> `Runtime::enableCoroutine(false)`は直前に設定されたすべてのオプションのコルーチン`Hook`を無効にします。
