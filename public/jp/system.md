# Coroutine\System

`API`の関連するシステムのコルーチンラッパー。 このモジュールは`v4.4.6`の公式バージョン以降で利用可能です。 大半の`API`は`AIO`スレッドプールに基づいて実装されています。

！> `v4.4.6`より前のバージョンでは、`Co`エイリアスや`Swoole\Coroutine`を使用してください。たとえば、`Co::sleep`または`Swoole\Coroutine::sleep`といった具合です。  
`v4.4.6`以降のバージョンでは、公式で**お勧め**されるのは`Co\System::sleep`または`Swoole\Coroutine\System::sleep`を使用することです。  
この変更は名前空間を統一するためのものであり、同時に下位互換性も確保されています（つまり、`v4.4.6`以前のバージョンでの書き方も有効であり、変更は不要です）。
This is a heading and doesn't require translation.
### statvfs()

ファイルシステム情報を取得します。

!> Swooleバージョン >= v4.2.5 で利用可能

```php
Swoole\Coroutine\System::statvfs(string $path): array|false
```

  * **パラメータ** 

    * **`string $path`**
      * **機能**：マウントされているファイルシステムのディレクトリ【例`/`、dfや`mount -l`コマンドで取得できます】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **使用例**

    ```php
    Swoole\Coroutine\run(function () {
        var_dump(Swoole\Coroutine\System::statvfs('/'));
    });
    
    // array(11) {
    //   ["bsize"]=>
    //   int(4096)
    //   ["frsize"]=>
    //   int(4096)
    //   ["blocks"]=>
    //   int(61068098)
    //   ["bfree"]=>
    //   int(45753580)
    //   ["bavail"]=>
    //   int(42645728)
    //   ["files"]=>
    //   int(15523840)
    //   ["ffree"]=>
    //   int(14909927)
    //   ["favail"]=>
    //   int(14909927)
    //   ["fsid"]=>
    //   int(1002377915335522995)
    //   ["flag"]=>
    //   int(4096)
    //   ["namemax"]=>
    //   int(255)
    // }
    ```
### fread()

ファイルを読み取るコルーチン方式。

```php
Swoole\Coroutine\System::fread(resource $handle, int $length = 0): string|false
```

!> `v4.0.4`以下のバージョンでは、`STDIN`、`ソケット`などの非ファイルタイプの`stream`が`fread`メソッドをサポートしていません。このようなリソースに`fread`を使用しないでください。  
`v4.0.4`以上のバージョンでは、`fread`メソッドは非ファイルタイプの`stream`リソースをサポートし、内部では`stream`タイプに応じて`AIO`スレッドプールまたは[EventLoop](/learn?id=什么是eventloop)を使用します。

  * **パラメータ** 

    * **`resource $handle`**
      * **機能**：ファイルハンドル【`fopen`で開かれたファイルタイプの`stream`リソースである必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $length`**
      * **機能**：読み取る長さ【デフォルトは`0`で、ファイルのすべての内容を読み取ることを意味します】
      * **デフォルト値**：`0`
      * **その他の値**：なし

  * **戻り値** 

    * 読み取りに成功した場合は文字列内容、失敗した場合は`false`を返します

  * **使用例**  

    ```php
    $fp = fopen(__FILE__, "r");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fread($fp);
        var_dump($r);
    });
    ```
### fwrite()

ファイルにデータを書き込むためのコルーチン方式。

```php
Swoole\Coroutine\System::fwrite(resource $handle, string $data, int $length = 0): int|false
```

!> バージョン `v4.0.4` 未満では、`STDIN`や`Socket`などの非ファイルタイプの`stream`をサポートしていませんので、これらのリソースで`fwrite`を使用しないでください。  
バージョン `v4.0.4` 以降、`fwrite`メソッドは非ファイルタイプの`stream`リソースをサポートしましたが、内部では`stream`のタイプに応じて`AIO`スレッドプールまたは[EventLoop](/learn?id=什么是eventloop)を自動選択します。

  * **パラメータ** 

    * **`resource $handle`**
      * **目的**：ファイルハンドル【`fopen`で開かれたファイルタイプの`stream`リソースである必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **目的**：書き込むデータの内容【テキストまたはバイナリデータが可能です】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $length`**
      * **目的**：読み取る長さ【デフォルトは`0`で、`$data`の全内容を書き込みます。`$length`は`$data`の長さよりも小さくする必要があります】
      * **デフォルト値**：`0`
      * **その他の値**：なし

  * **戻り値** 

    * 書き込みが成功した場合はデータの長さ、失敗した場合は`false`を返します。

  * **使用例**  

    ```php
    $fp = fopen(__DIR__ . "/test.data", "a+");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fwrite($fp, "hello world\n", 5);
        var_dump($r);
    });
    ```
### fgets()

ファイルの内容を行ごとに読み取るためのコルーチン方式。

内部では`php_stream`のバッファリングが使用されていて、デフォルトサイズは`8192`バイトです。`stream_set_chunk_size`を使用してバッファリングサイズを設定できます。

```php
Swoole\Coroutine\System::fgets(resource $handle): string|false
```

!> `fgets`関数はファイルタイプの`stream`リソースでのみ使用可能であり、Swooleバージョン >= `v4.4.4` で使用可能です

  * **Parameters** 

    * **`resource $handle`**
      * **Description**：ファイルハンドル【`fopen`で開いたファイルタイプの`stream`リソースである必要があります】
      * **Default**：なし
      * **Other values**：なし

  * **Return Value** 

    * `EOL`（`\r`または`\n`）を読み取れば、`EOL`を含む1行のデータが返されます
    * `EOL`が読み取れない場合でも内容の長さが`php_stream`のバッファリングサイズである8192バイトを超えた場合、8192バイトのデータが返されますが、`EOL`は含まれません
    * ファイルの末尾`EOF`に到達した場合、空の文字列が返されます。ファイルの終わりまで読み取ったかどうかを確認するには`feof`を使用できます
    * 読み取りに失敗した場合は`false`が返され、エラーコードを取得するには[swoole_last_error](/functions?id=swoole_last_error)関数を使用してください

  * **Usage Example**  

    ```php
    $fp = fopen(__DIR__ . "/defer_client.php", "r");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fgets($fp);
        var_dump($r);
    });
    ```  
### readFile()

ファイルをコルーチン方式で読み込みます。

```php
Swoole\Coroutine\System::readFile(string $filename): string|false
```

  * **パラメータ** 

    * **`string $filename`**
      * **機能**：ファイル名
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

    * 読み込みに成功した場合は文字列内容を返し、失敗した場合は`false`を返します。エラー情報は[swoole_last_error](/functions?id=swoole_last_error)を使用して取得できます。
    * `readFile`メソッドにはサイズ制限がありませんが、読み取られたコンテンツはメモリに保存されるため、非常に大きなファイルを読み取る場合は多くのメモリを使用する可能性があります。

  * **使用例**  

    ```php
    $filename = __DIR__ . "/defer_client.php";
    Swoole\Coroutine\run(function () use ($filename)
    {
        $r = Swoole\Coroutine\System::readFile($filename);
        var_dump($r);
    });
    ```
### writeFile()

ファイルに書き込むためのコルーチン方式です。

```php
Swoole\Coroutine\System::writeFile(string $filename, string $fileContent, int $flags): bool
```

  * **パラメータ** 

    * **`string $filename`**
      * **機能**：ファイル名【書き込み権限が必要であり、ファイルが存在しない場合は自動的に作成されます。ファイルのオープンに失敗するとすぐに`false`が返ります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $fileContent`**
      * **機能**：ファイルに書き込まれる内容【最大サイズは`4M`まで】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $flags`**
      * **機能**：書き込みオプション【通常は現在のファイル内容を消去し、`FILE_APPEND`を使用してファイルの末尾に追加することができます】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

    * 書き込みに成功した場合は`true`を返します
    * 書き込みに失敗した場合は`false`を返します

  * **使用例**  

    ```php
    $filename = __DIR__ . "/defer_client.php";
    Swoole\Coroutine\run(function () use ($filename)
    {
        $w = Swoole\Coroutine\System::writeFile($filename, "hello swoole!");
        var_dump($w);
    });
    ```  
### sleep()

ウェイティング状態に入ります。

`PHP`の`sleep`関数に相当し、`Coroutine::sleep`は[協力的スケジューリング](/coroutine?id=協調的スケジューリング)で実装されていることが異なる点です。内部では現在のコルーチンを`yield`し、時間を譲り、非同期タイマーを追加します。タイムアウトが発生すると、現在のコルーチンを再度`resume`して実行を再開します。

`sleep`インタフェースを使用すると、タイムアウト待ちなどの機能を簡単に実装できます。

```php
Swoole\Coroutine\System::sleep(float $seconds): void
```

  * **パラメーター**

    * **`float $seconds`**
      * **機能**: 眠る時間【`0`より大きい必要があり、1日（`86400`秒）を超えてはいけません】
      * **値の単位**：秒、最小の精度はミリ秒（`0.001`秒）
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **使用例**

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9502);

    $server->on('Request', function($request, $response) {
        // 200ms待ってからブラウザに応答を返す
        Swoole\Coroutine\System::sleep(0.2);
        $response->end("<h1>Hello Swoole!</h1>");
    });

    $server->start();
    ```
### exec()

シェルコマンドを実行します。内部的には[コルーチンスケジューリング](/coroutine?id=协程调度)が自動的に行われます。

```php
Swoole\Coroutine\System::exec(string $cmd): array
```

  * **パラメーター** 

    * **`string $cmd`**
      * **機能**：実行する`shell`コマンド
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値**

    * 実行に失敗した場合は`false`が返され、成功した場合はプロセスの終了ステータス、シグナル、出力内容を含む配列が返されます。

    ```php
    array(
        'code'   => 0,  // プロセスの終了ステータス
        'signal' => 0,  // シグナル
        'output' => '', // 出力内容
    );
    ```

  * **使用例**  

    ```php
    Swoole\Coroutine\run(function() {
        $ret = Swoole\Coroutine\System::exec("md5sum ".__FILE__);
    });
    ```

  * **注意**

  !>スクリプトコマンドの実行時間が長すぎるとタイムアウトが発生し、このような場合は[socket_read_timeout](/coroutine_client/init?id=超时规则)を増やすことでこの問題を解決できます。
### gethostbyname()

IPへのドメイン名の解決。 同期的なスレッドプールに基づくシミュレーション実装、アンダーレイヤーで自動的に[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)を行います。

```php
Swoole\Coroutine\System::gethostbyname(string $domain, int $family = AF_INET, float $timeout = -1): string|false
```

  * **Arguments**

    * **`string $domain`**
      * **Description**: ドメイン名
      * **Default**: なし
      * **Other Values**: なし

    * **`int $family`**
      * **Description**: ファミリ【`AF_INET`は`IPv4`アドレスを返し、`AF_INET6`を使用すると`IPv6`アドレスを返します】
      * **Default**: `AF_INET`
      * **Other Values**: `AF_INET6`

    * **`float $timeout`**
      * **Description**: タイムアウト時間
      * **Unit**: 秒、最小精度はミリ秒（`0.001`秒）
      * **Default**: `-1`
      * **Other Values**: なし

  * **Returns**

    * 成功した場合はドメインに対応するIPアドレスが返され、失敗した場合は`false`が返されます。 エラー情報は[swoole_last_error](/functions?id=swoole_last_error)を使用して取得できます

    ```php
    array(
        'code'   => 0,  // プロセスの終了ステータスコード
        'signal' => 0,  // シグナル
        'output' => '', // 出力内容
    );
    ```

  * **Extensions**

    * **タイムアウト制御**

      `timeout`パラメータは、コルーチンが待機するタイムアウト時間を制御します。 指定された時間内に結果が返らない場合、コルーチンはすぐに`false`を返し、以降の処理を続行します。 アンダーレイヤー実装では、この非同期タスクを`cancel`としてマークし、`gethostbyname`は引き続き`AIO`スレッドプールで実行されます。
      
      `/etc/resolv.conf`を変更して、`gethostbyname`および`getaddrinfo`のアンダーレイヤーの`C`関数のタイムアウト時間を設定できます。 詳細については[DNS解決のタイムアウトと再試行を設定する](/learn_other?id=セットdns解析タイムアウトと再試行)

  * **Usage Example**  

    ```php
    Swoole\Coroutine\run(function () {
        $ip = Swoole\Coroutine\System::gethostbyname("www.baidu.com", AF_INET, 0.5);
        echo $ip;
    });
    ```
### getaddrinfo()

DNS解析を実行し、ドメインに対応する`IP`アドレスを取得します。

`gethostbyname`とは異なり、`getaddrinfo`はより多くのパラメータ設定をサポートし、複数の`IP`結果を返します。

```php
Swoole\Coroutine\System::getaddrinfo(string $domain, int $family = AF_INET, int $socktype = SOCK_STREAM, int $protocol = STREAM_IPPROTO_TCP, string $service = null, float $timeout = -1): array|false
```

  * **Parameters** 

    * **`string $domain`**
      * **Function**：ドメイン
      * **Default**：なし
      * **Other values**：なし

    * **`int $family`**
      * **Function**：ファミリー【`AF_INET`は`IPv4`アドレスを返し、`AF_INET6`を使用すると`IPv6`アドレスを返します】
      * **Default**：なし
      * **Other values**：なし
      
      !> その他のパラメータ設定については、`man getaddrinfo`ドキュメントを参照してください

    * **`int $socktype`**
      * **Function**：ソケットタイプ
      * **Default**：`SOCK_STREAM`
      * **Other values**：`SOCK_DGRAM`、`SOCK_RAW`

    * **`int $protocol`**
      * **Function**：プロトコル
      * **Default**：`STREAM_IPPROTO_TCP`
      * **Other values**：`STREAM_IPPROTO_UDP`、`STREAM_IPPROTO_STCP`、`STREAM_IPPROTO_TIPC`、`0`

    * **`string $service`**
      * **Function**：
      * **Default**：なし
      * **Other values**：なし

    * **`float $timeout`**
      * **Function**：タイムアウト
      * **Unit**：秒、最小精度はミリ秒（`0.001`秒）
      * **Default**：`-1`
      * **Other values**：なし

  * **Return Value**

    * 成功：複数の`IP`アドレスから成る配列を返す。失敗：`false` を返す。

  * **Usage Example**  

    ```php
    Swoole\Coroutine\run(function () {
        $ips = Swoole\Coroutine\System::getaddrinfo("www.baidu.com");
        var_dump($ips);
    });
    ```
### dnsLookup()

ドメイン名のアドレス検索。

`Coroutine\System::gethostbyname`とは異なり、`Coroutine\System::dnsLookup`は`UDP`クライアントネットワーク通信に直接基づいて実装されており、`libc`の`gethostbyname`関数を使用していません。

!> Swoole バージョン >= `v4.4.3` で利用可能であり、基盤として`/etc/resolve.conf`を読み取って`DNS`サーバーアドレスを取得します。現時点では`AF_INET(IPv4)`ドメイン解決のみがサポートされています。Swoole バージョン >= `v4.7` では、`AF_INET6(IPv6)`をサポートするために第三引数を使用できます。

```php
Swoole\Coroutine\System::dnsLookup(string $domain, float $timeout = 5, int $type = AF_INET): string|false
```

  * **パラメータ** 

    * **`string $domain`**
      * **説明**：ドメイン名
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **説明**：タイムアウト時間
      * **単位**：秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`5`
      * **その他の値**：なし

    * **`int $type`**
      * **単位**：秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`AF_INET`
      * **その他の値**：`AF_INET6`

    !> `$type`引数は Swoole バージョン >= `v4.7` で利用可能です。

  * **戻り値**

    * 解決に成功した場合は対応するIPアドレスが返されます
    * 失敗した場合は`false`が返され、エラー情報は[swoole_last_error](/functions?id=swoole_last_error)を使用して取得できます

  * **一般的なエラー**

    * `SWOOLE_ERROR_DNSLOOKUP_RESOLVE_FAILED`：このドメイン名は解決できません、クエリが失敗しました
    * `SWOOLE_ERROR_DNSLOOKUP_RESOLVE_TIMEOUT`：解決がタイムアウトしました、DNSサーバーに障害が発生しており、指定された時間内に結果を返すことができません

  * **使用例**  

    ```php
    Swoole\Coroutine\run(function () {
        $ip = Swoole\Coroutine\System::dnsLookup("www.baidu.com");
        echo $ip;
    });
    ```
### wait()

[Process::wait](/process/process?id=wait)に対応していますが、このAPIはコルーチンバージョンであり、コルーチンを中断させる可能性があります。 `Swoole\Process::wait` および `pcntl_wait` 関数と置き換えることができます。

!> Swooleバージョン >= `v4.5.0` で利用可能

```php
Swoole\Coroutine\System::wait(float $timeout = -1): array|false
```

* **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間、負数はタイムアウトしないことを意味します
      * **単位**：秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`-1`
      * **その他の値**：なし

* **戻り値**

  * 成功すると、子プロセスの`PID`、終了ステータスコード、どのシグナルで`KILL`されたかを含む配列が返されます
  * 失敗すると`false`が返されます

!> 各子プロセスが起動した後、親プロセスはすべてのコルーチンを派遣して`wait()`（または`waitPid()`）を呼び出して回収する必要があります。そうしないと、子プロセスはゾンビプロセスになり、オペレーティングシステムのプロセスリソースが無駄になります。  
コルーチンを使用する場合は、プロセスを作成してからコルーチンを開始する必要があります。逆にしないでください。そうしないと、コルーチンを伴ったforkの状況が複雑になり、低レベルの処理が非常に困難になります。

* **例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    echo 'Hello Swoole';
});
$process->start();

Coroutine\run(function () use ($process) {
    $status = System::wait();
    assert($status['pid'] === $process->pid);
    var_dump($status);
});
```
### waitPid()

和上述wait方法基本一致, 不同的是此API可以指定等待特定的进程

!> Swoole版本 >= `v4.5.0` 可用

```php
Swoole\Coroutine\System::waitPid(int $pid, float $timeout = -1): array|false
```

* **参数** 

    * **`int $pid`**
      * **功能**：进程id
      * **默认值**：`-1` (表示任意进程, 此时等价于wait方法)
      * **其它值**：任意自然数

    * **`float $timeout`**
      * **功能**：超时时间，负数表示永不超时
      * **值单位**：秒，最小精度为毫秒（`0.001`秒）
      * **默认值**：`-1`
      * **其它值**：无

* **返回值**

  * 操作成功会返回一个数组包含子进程的`PID`、退出状态码、被哪种信号`KILL`
  * 失败返回`false`

!> 每个子进程启动后，父进程必须都要派遣一个协程调用`wait()`(或`waitPid()`)进行回收，否则子进程会变成僵尸进程，会浪费操作系统的进程资源。

* **示例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    echo 'Hello Swoole';
});
$process->start();

Coroutine\run(function () use ($process) {
    $status = System::waitPid($process->pid);
    var_dump($status);
});
```
```php
Swoole\Coroutine\System::waitSignal(int $signo, float $timeout = -1): bool
```

  * **参数** 

    * **`int $signo`**
      * **機能**：シグナルの種類
      * **デフォルト値**：なし
      * **その他の値**：`SIGTERM`、`SIGKILL`などのSIGシリーズ定数

    * **`float $timeout`**
      * **機能**：タイムアウト時間、負数はタイムアウトしないことを示す
      * **値の単位**：秒で、最小の精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`-1`
      * **その他の値**：なし

  * **返り値**

    * シグナルを受信した場合は`true`
    * タイムアウトしてシグナルを受信しなかった場合は`false`

  * **例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    Coroutine\run(function () {
        $bool = System::waitSignal(SIGUSR1);
        var_dump($bool);
    });
});
$process->start();
sleep(1);
$process::kill($process->pid, SIGUSR1);
```
### waitEvent()

`waitEvent()`は、イベントが発生するまで現在のCoroutineをブロックするコルーチンバージョンのシグナルリスナーです。IOイベントを待ち受け、`swoole_event`関連の関数と置き換えることができます。

!> Swooleのバージョン >= `v4.5` で利用可能

```php
Swoole\Coroutine\System::waitEvent(mixed $socket, int $events = SWOOLE_EVENT_READ, float $timeout = -1): int | false
```

* **パラメータ** 

    * **`mixed $socket`**
      * **機能**：ファイル記述子（ソケットオブジェクト、リソースなど、fdに変換可能な任意のタイプ）
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $events`**
      * **機能**：イベントタイプ
      * **デフォルト値**：`SWOOLE_EVENT_READ`
      * **その他の値**：`SWOOLE_EVENT_WRITE` または `SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE`

    * **`float $timeout`**
      * **機能**：タイムアウト時間、負数はタイムアウトしないことを示す
      * **単位**：秒、最小の精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`-1`
      * **その他の値**：なし

* **戻り値**

  * トリガーされたイベントのタイプと（複数のビットになる可能性があります）を返します。これは引数`$events`に関連しています。
  * 失敗した場合は`false`を返し、エラー情報は[swoole_last_error](/functions?id=swoole_last_error)を使用して取得できます。

* **例**

> 同期ブロッキングコードをこのAPIを使って非同期非ブロッキングに変換できます。

```php
use Swoole\Coroutine;

Coroutine\run(function () {
    $client = stream_socket_client('tcp://www.qq.com:80', $errno, $errstr, 30);
    $events = Coroutine::waitEvent($client, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE);
    assert($events === SWOOLE_EVENT_WRITE);
    fwrite($client, "GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");
    $events = Coroutine::waitEvent($client, SWOOLE_EVENT_READ);
    assert($events === SWOOLE_EVENT_READ);
    $response = fread($client, 8192);
    echo $response;
});
```
