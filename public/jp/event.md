# イベント

`Swoole`拡張機能には、バックエンドの`epoll/kqueue`イベントループを直接操作するためのインターフェイスが用意されています。他の拡張機能が作成した`socket`や、`PHP`コード内で`stream/socket`拡張機能で作成した`socket`を`Swoole`の[EventLoop](/learn?id=什么是eventloop)に追加できます。
さもなければ、第三者の$fdが同期IOである場合には、SwooleのEventLoopが実行されないことがあります。[参考案例](/learn?id=同步io转换成异步io)。

!> `Event`モジュールは比較的低レベルであり、`epoll`の基本的なラッパーです。利用者はIO多重化プログラミングの経験を持っていると良いでしょう。
## イベントの優先順位

1. `Process::signal`で設定されたシグナル処理コールバック関数
2. `Timer::tick`および`Timer::after`で設定されたタイマーコールバック関数
3. `Event::defer`で設定された遅延実行関数
4. `Event::cycle`で設定された周期コールバック関数
このセクションでは、さまざまな方法を紹介します。

```
def greet(name):
    return "Hello, " + name + "!"
```

次のセクションに進む前に、この方法を試してみてください。
### add()

`socket`を低レベルの`reactor`イベントリスナーに追加します。この関数は`Server`または`Client`モードで使用できます。
```php
Swoole\Event::add(mixed $sock, callable $read_callback, callable $write_callback = null, int $flags = null): bool
```

!> `Server` プログラムで使用する場合、`Worker` プロセスが起動した後に使用する必要があります。`Server::start` の前には非同期の `IO` インターフェースを呼び出してはいけません

* **パラメータ** 

  * **`mixed $sock`**
    * **説明**：ファイル記述子、`stream`リソース、`sockets`リソース、オブジェクト
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $read_callback`**
    * **説明**：読み込み可能なイベントのコールバック関数
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $write_callback`**
    * **説明**：書き込み可能なイベントのコールバック関数【このパラメータは文字列の関数名、オブジェクト+メソッド、クラスの静的メソッド、無名関数のいずれかにすることができます。この `socket` が読み取り可能または書き込み可能になったときに指定された関数がコールバックされます。】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $flags`**
    * **説明**：イベントの種類のマスク【読み込み可能なイベント、書き込み可能なイベントを閉じる/オープンすることができます。例：`SWOOLE_EVENT_READ`、`SWOOLE_EVENT_WRITE`または`SWOOLE_EVENT_READ|SWOOLE_EVENT_WRITE`】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **$sock の4つの型**

タイプ | 説明
---|---
int | `Swoole\Client->$sock`、`Swoole\Process->$pipe`などのファイル記述子
streamリソース | `stream_socket_client`/`fsockopen` で作成されたリソース
socketsリソース | `sockets`拡張で作成されたリソース。[./configure --enable-sockets](/environment?id=コンパイルオプション) でコンパイルする必要があります。
object | `Swoole\Process`または`Swoole\Client`。自動的に[UnixSocket](/learn?id=IPCとは)（`Process`）またはクライアント接続の `socket`（`Swoole\Client`）に変換されます。

* **戻り値**

  * イベントリスナーの追加が成功した場合は `true` が返されます
  * 失敗した場合は `false` が返されます。エラーコードを取得するには `swoole_last_error` を使用してください
  * 既に追加された `socket` は再度追加できません。 `swoole_event_set` を使用して `socket` に関連するコールバック関数とイベントタイプを変更できます

  !> `Swoole\Event::add` を使用して `socket` をイベントリスナーに追加すると、この `socket` は自動的にノンブロッキングモードに設定されます

* **使用例**

```php
$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

Swoole\Event::add($fp, function($fp) {
    $resp = fread($fp, 8192);
    //ソケット処理が完了した後に、epoll イベントからソケットを削除する
    Swoole\Event::del($fp);
    fclose($fp);
});
echo "Finish\n";  //Swoole\Event::add はプロセスをブロックしません。この行のコードは順次実行されます
```

* **コールバック関数**

  * 読み込み可能な`($read_callback)` イベントコールバック関数では、`fread`、`recv`などの関数を使用して `socket` のバッファ内のデータを読み取らなければならず、それ以外の場合はイベントが継続的にトリガーされます。継続的に読み取りを行いたくない場合は `Swoole\Event::del` を使用してイベントリスナーを削除する必要があります
  * 書き込み可能な`($write_callback)` イベントコールバック関数では、`socket` への書き込み後、`Swoole\Event::del` を呼び出してイベントリスナーを削除する必要があります。 そうしないと書き込み可能なイベントが継続的にトリガーされます
  * `fread`、`socekt_recv`、`socket_read`、`Swoole\Client::recv` を実行した際に `false` が返り、かつエラーコードが `EAGAIN` の場合は、現在の `socket` の受信バッファ内にデータがないことを意味し、読み込みリスナーを待機する必要があります。[EventLoop](/learn?id=eventloop) 通知があります
  * `fwrite`、`write`、`socket_send`、`Swoole\Client::send` を実行した際に `false` が返り、かつエラーコードが `EAGAIN` の場合は、現在の `socket` の送信バッファがいっぱいでデータを送信できないことを意味します。書き込み可能なイベントをリスニングし、[EventLoop](/learn?id=eventloop) 通知を待機する必要があります。
### set()

イベントリスナーのコールバック関数とマスクを変更します。

```php
Swoole\Event::set($fd, mixed $read_callback, mixed $write_callback, int $flags): bool
```

* **パラメータ**

  * パラメータは [Event::add](/event?id=add) とまったく同じです。`$fd` が存在しない場合は `false` を返します。
  * `$read_callback` が `null` でない場合、指定された関数に読み取り可能なイベントコールバック関数を変更します。
  * `$write_callback` が `null` でない場合、指定された関数に書き込み可能なイベントコールバック関数を変更します。
  * `$flags` は、読み取り（`SWOOLE_EVENT_READ`）および書き込み（`SWOOLE_EVENT_WRITE`）イベントのリスニングをオン/オフにできます。

  !> `SWOOLE_EVENT_READ` イベントをリスンしていて現在 `read_callback` が設定されていない場合、基盤は直接 `false` を返し、追加に失敗します。`SWOOLE_EVENT_WRITE` も同様です。

* **状態変更**

  * 読み取り可能なイベントコールバックを設定するために `Event::add` または `Event::set` を使用していますが、`SWOOLE_EVENT_READ` 読み取りイベントをリスンしていません。この場合、基盤はコールバック関数の情報のみを保存し、イベントコールバックは生成されません。
  * `Event::set($fd, null, null, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)` を使用して、リスンするイベントタイプを変更することがでいきます。この場合、基盤は読み取り可能なイベントをトリガーします。

* **コールバック解放**

  !> `Event::set` はコールバック関数を置き換えるだけであり、イベントコールバック関数を解放することはできません。例：`Event::set($fd, null, null, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)` という引数を渡すと、`read_callback` と `write_callback` は `null` になり、 `Event::add` に設定されたコールバック関数を変更しないことを意味します。

`Event::del` を使用してイベントリスニングを解除しない限り、基盤は `read_callback` および `write_callback` イベントコールバック関数を解放しません。
### isset()

`$fd`がイベントリスナーに追加されたかを検出します。

```php
Swoole\Event::isset(mixed $fd, int $events = SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE): bool
```

* **パラメーター**

  * **`mixed $fd`**
    * **機能**：任意のソケットファイルディスクリプタ【参考：[Event::add](/event?id=add) ドキュメント】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $events`**
    * **機能**：検出するイベントタイプ
    * **デフォルト値**：なし
    * **その他の値**：なし

* **$events**

イベントタイプ | 説明
---|---
`SWOOLE_EVENT_READ` | 読み取り可能なイベントをリスニングしているかどうか
`SWOOLE_EVENT_WRITE` | 書き込み可能なイベントをリスニングしているかどうか
`SWOOLE_EVENT_READ \| SWOOLE_EVENT_WRITE` | 読み取り可能または書き込み可能なイベントをリスニングしているかどうか

* **使用例**

```php
use Swoole\Event;

$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

Event::add($fp, function($fp) {
    $resp = fread($fp, 8192);
    Swoole\Event::del($fp);
    fclose($fp);
}, null, SWOOLE_EVENT_READ);
var_dump(Event::isset($fp, SWOOLE_EVENT_READ)); // trueを返します
var_dump(Event::isset($fp, SWOOLE_EVENT_WRITE)); // falseを返します
var_dump(Event::isset($fp, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)); // trueを返します
```
### write()

PHPの組み込みの`stream/sockets`拡張機能で作成されたソケットに対し、`fwrite/socket_send`などの関数を使用して対向端にデータを送信します。送信するデータ量が多い場合、ソケットの書き込みバッファがいっぱいになると、送信ブロックまたは[EAGAIN](/other/errno?id=linux)エラーが発生します。

`Event::write`関数は`stream/sockets`リソースのデータ送信を非同期に変換します。バッファがいっぱいになったり[EAGAIN](/other/errno?id=linux)が返された場合、Swooleの内部はそのデータを送信キューに追加し、書き込み可能になるのを監視します。ソケットが書き込み可能になった時、Swooleの内部で自動的に書き込まれます。

```php
Swoole\Event::write(mixed $fd, miexd $data): bool
```

* **パラメーター** 

  * **`mixed $fd`**
    * **説明**：任意のソケットファイルディスクリプタ【参照：[Event::add](/event?id=add) ドキュメント】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`miexd $data`**
    * **説明**：送信するデータ 【データの長さは`Socket`バッファサイズを超えてはいけません】
    * **デフォルト値**：なし
    * **その他の値**：なし

!> `SSL/TLS`などのトンネル暗号化が使われている`stream/sockets`リソースには`Event::write`を使用できません  
`Event::write`が成功すると、その`$socket`が自動的にノンブロッキングモードに設定されます。

* **使用例**

```php
use Swoole\Event;

$fp = stream_socket_client('tcp://127.0.0.1:9501');
$data = str_repeat('A', 1024 * 1024*2);

Event::add($fp, function($fp) {
     echo fread($fp);
});

Event::write($fp, $data);
```
#### SOCKET缓存区がいっぱいになった場合のSwooleのロジック

`SOCKET`に持続的に書き込みが行われ、対向端が読み取り速度が遅い場合、`SOCKET`のキャッシュ領域が満杯になります。`Swoole`の基盤は、データをメモリキャッシュ領域に保存し、書き込み可能なイベントがトリガーされるまで`SOCKET`に書き込みます。

もしメモリキャッシュ領域もいっぱいになった場合、この時`Swoole`の基盤は`pipe buffer overflow, reactor will block.`エラーをスローし、ブロック待機状態に入ります。

!> キャッシュがいっぱいになった場合、`false`が返り、これは原子操作です。すべての書き込みが成功するか、すべて失敗するかのどちらかのみが発生します。
```php
Swoole\Event::del(mixed $sock): bool
```

!> `Event::del`を使用して、`socket`のイベントリスナーを削除する前に`socket`を`close`する必要があります。そうしないと、メモリリークが発生する可能性があります

* **パラメーター** 

  * **`mixed $sock`**
    * **説明**：`socket`のファイルディスクリプタ
    * **デフォルト値**：なし
    * **その他の値**：なし
### exit()

Exit the event loop.

!> This function is only effective in the `Client` program.

```php
Swoole\Event::exit(): void
```
### defer()

次のイベントループの開始時に関数を実行します。

```php
Swoole\Event::defer(mixed $callback_function);
```

!> `Event::defer`のコールバック関数は、現在の`EventLoop`のイベントループが終了した後、次のイベントループが開始する前に実行されます。

* **パラメータ** 

  * **`mixed $callback_function`**
    * **機能**：時間が経過した後に実行される関数【呼び出し可能である必要があります。コールバック関数は引数を受け取らず、匿名関数の`use`構文を使用して引数をコールバック関数に渡すことができます；`$callback_function`関数の実行中に新しい`defer`タスクを追加しても、そのタスクは引き続き同じイベントループ内で完了します】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **使用例**

```php
Swoole\Event::defer(function(){
    echo "After EventLoop\n";
});
```
### cycle()

イベントループの周期的な実行関数を定義します。この関数は、各イベントループの終了時に呼び出されます。

```php
Swoole\Event::cycle(callable $callback, bool $before = false): bool
```

* **パラメータ**

  * **`callable $callback_function`**
    * **機能**：設定するコールバック関数【`$callback`が`null`の場合、`cycle`関数をクリアします。既に設定されたcycle関数がある場合、新しく設定すると前回の設定が上書きされます】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`bool $before`**
    * **機能**：[EventLoop](/learn?id=什么是eventloop)の前にこの関数を呼び出します
    * **デフォルト値**：なし
    * **その他の値**：なし

!> `before=true`と`before=false`の2つのコールバック関数を同時に使用できます。

  * **使用例**

```php
Swoole\Timer::tick(2000, function ($id) {
    var_dump($id);
});

Swoole\Event::cycle(function () {
    echo "hello [1]\n";
    Swoole\Event::cycle(function () {
        echo "hello [2]\n";
        Swoole\Event::cycle(null);
    });
});
```  
### wait()

イベントリスナーを開始します。

!> この関数をPHPプログラムの末尾に配置してください

```php
Swoole\Event::wait();
```

* **使用例**

```php
Swoole\Timer::tick(1000, function () {
    echo "hello\n";
});

Swoole\Event::wait();
```
### dispatch()

イベントリスナーを起動します。

!> `reactor->wait`操作を一度だけ実行し、`Linux`プラットフォームでは`epoll_wait`を1回手動で呼び出すことになります。これは`Event::dispatch`とは異なり、`Event::wait`は内部でループを維持します。

```php
Swoole\Event::dispatch();
```

* **使用例**

```php
while(true)
{
    Event::dispatch();
}
```

この関数の目的は、`amp`などのフレームワークとの互換性を確保することです。このようなフレームワークでは、フレームワーク自体が`reactor`のループを制御しますが、`Event::wait`を使用すると、Swooleの基礎層が制御権を持つため、フレームワークにコントロールを渡すことができません。
