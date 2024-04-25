```
print("Hello, World!")
```

このコードは、"Hello, World!"というテキストをコンソールに出力します。
## Swoole性能はどうですか

> QPS比較

Nginxの静的ページ、GolangのHTTPプログラム、PHP7+SwooleのHTTPプログラムをApache Benchツール（ab）を使用してストレステストしました。同じマシンで、100の同時接続で100万回のHTTPリクエストの負荷テストを行った場合のQPS比較は次の通りです：

| ソフトウェア | QPS | ソフトウェアバージョン |
| --- | --- | --- |
| Nginx | 164489.92	| nginx/1.4.6 (Ubuntu) |
| Golang |	166838.68 |	go version go1.5.2 linux/amd64 |
| PHP7+Swoole |	287104.12 |	Swoole-1.7.22-alpha |
| Nginx-1.9.9 |	245058.70 |	nginx/1.9.9 |

!> 注：Nginx-1.9.9では、access_logを無効にし、open_file_cacheを使用して静的ファイルをメモリにキャッシュしています。

> テスト環境

* CPU：Intel® Core™ i5-4590 CPU @ 3.30GHz × 4
* メモリ：16G
* ディスク：128G SSD
* オペレーティングシステム：Ubuntu14.04 (Linux 3.16.0-55-generic)

> テスト手法

```shell
ab -c 100 -n 1000000 -k http://127.0.0.1:8080/
```

> VHOSTの設定

```nginx
server {
    listen 80 default_server;
    root /data/webroot;
    index index.html;
}
```

> テストページ

```html
<h1>Hello World!</h1>
```

> プロセス数

Nginxは4つのWorkerプロセスを起動しています
```shell
htf@htf-All-Series:~/soft/php-7.0.0$ ps aux|grep nginx
root      1221  0.0  0.0  86300  3304 ?        Ss   12月07   0:00 nginx: master process /usr/sbin/nginx
www-data  1222  0.0  0.0  87316  5440 ?        S    12月07   0:44 nginx: worker process
www-data  1223  0.0  0.0  87184  5388 ?        S    12月07   0:36 nginx: worker process
www-data  1224  0.0  0.0  87000  5520 ?        S    12月07   0:40 nginx: worker process
www-data  1225  0.0  0.0  87524  5516 ?        S    12月07   0:45 nginx: worker process
```

> Golang

テストコード

```go
package main

import (
    "log"
    "net/http"
    "runtime"
)

func main() {
    runtime.GOMAXPROCS(runtime.NumCPU() - 1)

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Add("Last-Modified", "Thu, 18 Jun 2015 10:24:27 GMT")
        w.Header().Add("Accept-Ranges", "bytes")
        w.Header().Add("E-Tag", "55829c5b-17")
        w.Header().Add("Server", "golang-http-server")
        w.Write([]byte("<h1>\nHello world!\n</h1>\n"))
    })

    log.Printf("Go http Server listen on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

> PHP7+Swoole

PHP7では`OPcache`アクセラレータが有効になっています。

テストコード

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501, SWOOLE_BASE);

$http->set([
    'worker_num' => 4,
]);

$http->on('request', function ($request, Swoole\Http\Server $response) {
    $response->header('Last-Modified', 'Thu, 18 Jun 2015 10:24:27 GMT');
    $response->header('E-Tag', '55829c5b-17');
    $response->header('Accept-Ranges', 'bytes');
    $response->end("<h1>\nHello Swoole.\n</h1>");
});

$http->start();
```

> **グローバルWebフレームワーク権威性能テスト TechEmpower Web Framework Benchmarks**

最新のベンチマーク結果はこちら: [techempower](https://www.techempower.com/benchmarks/#section=test&runid=9d5522a6-2917-467a-9d7a-8c0f6a8ed790)

Swooleは**ダイナミック言語の中でトップ**

データベースIO操作のテストは、特に最適化を加えていない基本ビジネスコードを使用しています。

**全ての静的言語フレームワークを凌駕する性能（PostgreSQLではなくMySQLを使用）**
## Swoole如何维持TCP长连接

TCP長いコネクションを維持するために、[tcp_keepalive](/server/setting?id=open_tcp_keepalive)と[heartbeat](/server/setting?id=heartbeat_check_interval)の2つの設定グループがあります。使用方法と注意事項については、[Swoole公式ビデオチュートリアル](https://course.swoole-cloud.com/course-video/10)を参照してください。
## Swooleの正しいサービス再起動方法

日常的な開発では、PHPコードを変更した後にコードを有効にするためにサービスを再起動する必要があります。繁忙なバックエンドサーバーは常にリクエストを処理しており、管理者が`kill`プロセスを使用してサーバープログラムを終了/再起動すると、コードが途中で停止する可能性があり、ビジネスロジックの完全性を保証できません。

`Swoole`は柔軟な終了/再起動機構を提供し、管理者は`Server`に特定のシグナルを送信したり`reload`メソッドを呼び出すだけで、ワーカープロセスを終了して再起動できます。詳細は[reload()](/server/methods?id=reload)を参照してください。

ただし、いくつかの注意点があります：

まず、新しく修正されたコードは`OnWorkerStart`イベントで再度読み込まれる必要があるため、例えばあるクラスが`OnWorkerStart`の前にcomposerのautoload経由で読み込まれている場合、効果がありません。

次に、`reload`はこれらの2つのパラメータ[max_wait_time](/server/setting?id=max_wait_time)と[reload_async](/server/setting?id=reload_async)と一緒に設定する必要があり、これらのパラメータを設定すると`非同期セーフ再起動`が可能になります。

この機能がない場合、Workerプロセスは再起信号を受け取るか[max_request](/server/setting?id=max_request)に達したときに、即座にサービスを停止します。この時、`Worker`プロセス内にはまだイベントリスナーがあり、これらの非同期タスクは破棄されます。上記のパラメータを設定すると、まず新しい`Worker`が作成され、古い`Worker`はすべてのイベントを完了した後に自動的に終了します（`reload_async`）。

古い`Worker`が終了しない場合、対応する時間([max_wait_time](/server/setting?id=max_wait_time)秒)内に強制的に終了され、[WARNING](/question/use?id=forced-to-terminate)エラーが発生します。

例：

```php
<?php
$serv = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS);
$serv->set(array(
    'worker_num' => 1,
    'max_wait_time' => 60,
    'reload_async' => true,
));
$serv->on('receive', function (Swoole\Server $serv, $fd, $reactor_id, $data) {

    echo "[#" . $serv->worker_id . "]\tClient[$fd] receive data: $data\n";
    
    Swoole\Timer::tick(5000, function () {
        echo 'tick';
    });
});

$serv->start();
```
### プロセス終了イベント

非同期の再起動機能をサポートするために、基盤には新しい[onWorkerExit](/server/events?id=onWorkerExit)イベントが追加されました。古い`Worker`が終了する直前に`onWorkerExit`イベントがトリガーされ、このイベントのコールバック関数内で、アプリケーション層は一部の長接続`Socket`をクリーンアップすることができます。`イベントループ`中にfdがなくなるか、[max_wait_time](/server/setting?id=max_wait_time)がプロセスを終了するまで、後者に達するまで。

```php
$serv->on('WorkerExit', function (Swoole\Server $serv, $worker_id) {
    $redisState = $serv->redis->getState();
    if ($redisState == Swoole\Redis::STATE_READY or $redisState == Swoole\Redis::STATE_SUBSCRIBE)
    {
        $serv->redis->close();
    }
});
```

また、[Swoole Plus](https://www.swoole.com/swoole_plus)では、ファイルの変更を自動的に検出する機能が追加され、手動でのリロードやシグナルの送信が不要になり、ファイルの変更でワーカーが自動的に再起動されます。
## なぜsend後すぐにcloseするのは安全でないのか

send後すぐにcloseするのは、サーバー側でもクライアント側でも安全ではありません。

send操作が成功したとしても、それはデータがオペレーティングシステムのソケットバッファに書き込まれたことを意味し、相手が実際にデータを受信したことを保証しません。オペレーティングシステムが送信したかどうか、相手のサーバーが受信したか、サーバー側のプログラムが処理したか、これらは確実に保証されません。

> close後のロジックについては、以下のlinger設定をご覧ください

このロジックは、電話でのコミュニケーションと同じです。AがBに何かを伝え、Aが話し終わったら電話を切る。その後、Bが聞いているかどうか、Aは分かりません。もしAが話し終わり、Bが了承し、その後Bが電話を切るなら、それは完全に安全です。

linger設定

`socket`がcloseされる際、バッファにまだデータがある場合、オペレーティングシステムの下で`linger`設定に応じて処理が行われます。

```c
struct linger
{
     int l_onoff;
     int l_linger;
};
```

- l_onoff = 0：close時にすぐにリターンし、未送信のデータが送信された後にリソースが解放され、優雅に終了します。
- l_onoff != 0 かつ l_linger = 0：close時にはすぐにリターンしますが、未送信のデータを送信せずに、RSTパケットを使用してソケット記述子を強制的に閉じます。
- l_onoff != 0 かつ l_linger > 0：close時にはすぐにリターンせず、カーネルは一定時間待ち、その時間はl_lingerの値で決まります。タイムアウト前に未送信のデータ（FINパケットを含む）を送信し、他端から確認を受けると、closeは正常にリターンし、ソケット記述子は優雅に終了します。それ以外の場合、closeは直ちにエラーを返し、未送信のデータが失われ、ソケット記述子が強制的に終了します。非ブロッキングモードに設定されている場合、closeは直ちに値を返します。
## 別のコルーチンにすでにバインドされたクライアント

`TCP`接続に対して、Swooleの内部では同時に1つのコルーチンが読み取り操作を行い、もう1つのコルーチンが書き込み操作を行うことができます。つまり、複数のコルーチンが同じTCP接続に対して読み取り/書き込み操作をすることはできず、内部でバインドエラーが発生します：

```shell
Fatal error: Uncaught Swoole\Error: Socket#6 has already been bound to another coroutine#2, reading or writing of the same socket in coroutine#3 at the same time is not allowed 
```

再現するコード：

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function() {
    $cli = new Client('www.xinhuanet.com', 80);
    Coroutine::create(function () use ($cli) {
        $cli->get('/');
    });
    Coroutine::create(function () use ($cli) {
        $cli->get('/');
    });
});
```

解決策は以下を参照してください：https://wenda.swoole.com/detail/107474

!> この制限はすべてのマルチコルーチン環境で有効であり、最も一般的なのは[onReceive](/server/events?id=onreceive)などのコールバック関数で1つのTCP接続を共有する場合です。なぜなら、この種のコールバック関数は自動的にコルーチンを作成するからです。接続プールが必要な場合はどうすればよいですか？`Swoole`には直接使用できる[接続プール](/coroutine/conn_pool)が組み込まれているか、またはマニュアルで`channel`を使用して接続プールをラップすることができます。
本文書のほとんどの例では、`Co\run()`を使用してコルーチンコンテナを作成します。[コルーチンコンテナとは何か](/coroutine?id=什么是协程容器)

以下のようなエラーが発生した場合：

```bash
PHP Fatal error:  Uncaught Error: Call to undefined function Co\run()

PHP Fatal error:  Uncaught Error: Call to undefined function go()
```

`Swoole`拡張機能のバージョンが`v4.4.0`未満であるか、[コルーチン短縮名](/other/alias?id=协程短名称)が手動で無効になっている可能性があります。次の解決方法を提供します。

- バージョンが古い場合は、拡張機能のバージョンを`>= v4.4.0`にアップグレードするか、コルーチンを作成するために`go`キーワードで`Co\run`を置換してください；
- コルーチン短縮名が無効になっている場合は、[コルーチン短縮名](/other/alias?id=协程短名称)を有効にしてください；
- [Coroutine::create](/coroutine/coroutine?id=create)メソッドを使用して`Co\run`または`go`を置き換えてコルーチンを作成してください；
- 完全修飾名を使用する：`Swoole\Coroutine\run`；
## RedisまたはMySQL接続を共有できますか

絶対にできません。各プロセスは`Redis`、`MySQL`、`PDO`接続を個別に作成する必要があります。他のストレージクライアントも同様です。1つの接続を共有すると、結果がどのプロセスで処理されるか保証されず、接続を保持しているプロセスは理論的にその接続を読み書きできるため、データが混乱します。

**したがって、複数のプロセス間で接続を共有してはいけません**

* [Swoole\Server](/server/init)では、[onWorkerStart](/server/events?id=onworkerstart)で接続オブジェクトを作成する必要があります。
* [Swoole\Process](/process/process)では、[Swoole\Process->start](/process/process?id=start)後に、子プロセスのコールバック関数で接続オブジェクトを作成する必要があります。
* この問題の情報は`pcntl_fork`を使用しているプログラムにも同様に適用されます。

例：

```php
$server = new Swoole\Server('0.0.0.0', 9502);

// 必ずonWorkerStartコールバック内でredis/mysql接続を作成する
$server->on('workerstart', function($server, $id) {
    $redis = new Redis();
	$redis->connect('127.0.0.1', 6379);
	$server->redis = $redis;
});

$server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {	
	$value = $server->redis->get("key");
	$server->send($fd, "Swoole: ".$value);
});

$server->start();
```
## 接続が閉じられた問題

以下の通知のように

```bash
NOTICE swFactoryProcess_finish (ERRNO 1004): send 165 byte failed, because connection[fd=123] is closed

NOTICE swFactoryProcess_finish (ERROR 1005): connection[fd=123] does not exists
```

サーバーの応答時、クライアントが接続を切断しているため

一般的な原因は：

- ブラウザがページを繰り返しリフレッシュする（まだ読み込み中にリフレッシュされる）
- ab による負荷テストが途中でキャンセルされる
- 時間に基づいた wrk テスト（時間切れになった未完了のリクエストはキャンセルされる）

これらの状況はすべて正常な現象であり、無視してもかまいません。したがって、このエラーのレベルはNOTICEです。

他の理由で接続が突然切断されて大量の場合のみ、注意が必要です

```bash
WARNING swWorker_discard_data (ERRNO 1007): [2] received the wrong data[21 bytes] from socket#75

WARNING Worker_discard_data (ERRNO 1007): [2] ignore data[5 bytes] received from session#2
```

同様に、このエラーも接続がすでに閉じられており、受信したデータは破棄されます。[discard_timeout_request](/server/setting?id=discard_timeout_request)を参照してください。
## connected property and connection status are inconsistent

4.x coroutine version
After, the `connected` property will no longer be updated in real time, and the [isConnect](/client?id=isconnected) method will no longer be reliable.
### 原因

協力の目標は、同期ブロッキングのプログラミングモデルと一致させることです。同期ブロッキングモデルでは、リアルタイムで接続の状態を更新する概念は存在しません。例えば、PDOやcurlのようなものには接続の概念はありません。IO操作時にエラーが発生するか、例外がスローされるまで接続が切断されていることに気づくことができます。

Swooleの底層では、IOエラー時にfalse（または空のコンテンツ）を返し、クライアントオブジェクトに対応するエラーコードとエラーメッセージを設定します。
### 注意

Even though the previous asynchronous version supported "real-time" updates to the `connected` attribute, it was not reliable in practice, as the connection could be lost immediately after you checked it.
## Connection refusedは何を意味しますか

telnet 127.0.0.1 9501を実行するとConnection refusedと表示される場合、サーバーがこのポートをリッスンしていないことを意味します。

* プログラムが正常に実行されているか確認する：ps aux
* ポートがリッスンされているか確認する：netstat -lp
* ネットワーク通信が正常かどうか確認する：tcpdump traceroute
## リソースが一時的に利用できません [11]

クライアントの`swoole_client`は`recv`中に次のエラーを通知します。

```shell
swoole_client::recv(): recv() failed. Error: Resource temporarily unavailable [11]
```

このエラーは、サーバーが指定された時間内にデータを返さず、タイムアウトしたことを示しています。

* ネットワーク通信のプロセスをtcpdumpで確認し、サーバーがデータを送信したかどうかをチェックできます。
* サーバーの`$serv->send`関数がtrueを返したかどうかを確認する必要があります。
* 外部ネットワーク通信時には、通信に時間がかかるため、`swoole_client`のタイムアウト時間を延長する必要があります。
以下のようなエラーが見つかりました：

```bash
WARNING swWorker_reactor_try_to_exit (ERRNO 9012): worker exit timeout, forced to terminate
```

これは、規定の時間内（[max_wait_time](/server/setting?id=max_wait_time)秒）にこの Worker が終了しなかったことを示し、Swoole の内部でプロセスを強制的に終了させました。

以下のコードを使用して再現することができます：

```php
use Swoole\Timer;

$server = new Swoole\Server('127.0.0.1', 9501);
$server->set(
    [
        'reload_async' => true,
        'max_wait_time' => 4,
    ]
);

$server->on('workerStart', function (Swoole\Server $server, int $wid) {
    if ($wid === 0) {
        Timer::tick(5000, function () {
            echo 'tick';
        });
        Timer::after(500, function () use ($server) {
            $server->shutdown();
        });
    }
});

$server->on('receive', function () {

});

$server->start();
```
## シグナル Broken pipe 用のコールバック関数が見つかりません: 13

以下のようなエラーが見つかりました：

```bash
WARNING swSignalfd_onSignal (ERRNO 707): Unable to find callback function for signal Broken pipe: 13
```

これは切断された接続にデータを送信したことを表しており、通常は送信の戻り値を確認していないため、失敗したまま送信を続けている可能性があります。
## 学習Swooleにはどの基本知識が必要ですか？
### 多プロセス/マルチスレッド

* `Linux`オペレーティングシステムのプロセスとスレッドの概念を理解する
* `Linux`でのプロセス/スレッドの切り替えに関する基本知識を持つ
* プロセス間通信の基本知識、例えばパイプ、`UnixSocket`、メッセージキュー、共有メモリを理解する
### ソケット

- `accept/connect`、`send/recv`、`close`、`listen`、`bind`など、**ソケット**の基本操作を理解しています。
- 受信バッファ、送信バッファ、ブロッキング/非ブロッキング、タイムアウトなど、**ソケット**の概念を理解しています。
### IO multiplexing

* 理解`select`/`poll`/`epoll`
* 理解基于`select`/`epoll`实现的事件循環, `Reactor` 模型
* 理解可讀事件、可寫事件
### TCP/IPネットワークプロトコル

- `TCP/IP`プロトコルの理解
- `TCP`、`UDP`転送プロトコルの理解
### デバッグツール

* [gdb](/other/tools?id=gdb)を使用して`Linux`プログラムをデバッグする
* [strace](/other/tools?id=strace)を使用してプロセスのシステムコールをトレースする
* [tcpdump](/other/tools?id=tcpdump)を使用してネットワーク通信をトレースする
* その他の`Linux`システムツール：ps、[lsof](/other/tools?id=lsof)、top、vmstat、netstat、sar、ssなど
## Swoole\Curl\Handler クラスのオブジェクトは int に変換できません

[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl) を使用中に次のエラーが発生しました：

```bash
PHP Notice:  Object of class Swoole\Curl\Handler could not be converted to int

PHP Warning: curl_multi_add_handle() expects parameter 2 to be resource, object given
```

原因は、hook後のcurlがリソースタイプではなくオブジェクトタイプであるため、intタイプに変換できないことです。

!> `int`の問題については、SDKチームに連絡してコードを変更することをお勧めします。PHP8では、curlはもはやリソースタイプではなくオブジェクトタイプになりました。

解決策は3つあります：

1. [SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)を有効にしない。ただし、[v4.5.4](/version/log?id=v454)バージョン以降、[SWOOLE_HOOK_ALL](/runtime?id=swoole_hook_all) にはデフォルトで[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)が含まれていますので、[SWOOLE_HOOK_ALL ^ SWOOLE_HOOK_CURL]と設定して[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)を無効にすることができます。

2. GuzzleのSDKを使用し、Handlerを置換してCoroutineを実現することができます。

3. Swoole `v4.6.0` バージョン以降では、[SWOOLE_HOOK_NATIVE_CURL](/runtime?id=swoole_hook_native_curl)を使用して[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl)を置き換えることができます。
## Guzzle 7.0+を使用して一括非同期化し、リクエストを送信した後に結果を直接端末に出力する方法 :id=hook_guzzle

再現コードは以下のとおりです。

```php
// composer require guzzlehttp/guzzle
include __DIR__ . '/vendor/autoload.php';

use GuzzleHttp\Client;
use Swoole\Coroutine;

// v4.5.4以前のバージョン
//Coroutine::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_CURL]);
Coroutine::set(['hook_flags' => SWOOLE_HOOK_ALL]);
Coroutine\run(function () {
    $client = new Client();
    $url = 'http://baidu.com';
    $res = $client->request('GET', $url);
    var_dump($res->getBody()->getContents());
});

// リクエストの結果が直接出力され、印刷されたものではありません
//<html>
//<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
//</html>
//string(0) ""
```

!> 問題の解決方法は前回のものと同じです。ただし、この問題は Swoole バージョン `v4.5.8` 以上で修正されています。
## Error: No buffer space available[55]

このエラーは無視できます。これは、[socket_buffer_size](/server/setting?id=socket_buffer_size) オプションが大きすぎて、一部のシステムで受け入れられないためであり、プログラムの実行には影響しません。
## GET/POSTリクエストの最大サイズ

Code blocks内のテキスト以外を日本語に翻訳しました。特定のコンテキストがある場合、その内容に基づいてお答えしますので、なんでもお気軽にお尋ねください。
### GETリクエストの最大8192

GETリクエストには1つのHTTPヘッダしかありません。Swooleの内部では、固定サイズの8Kメモリバッファを使用しており、このサイズは変更できません。正しいHTTPリクエストでない場合、エラーが発生します。内部で以下のエラーが発生します：

```bash
WARN swReactorThread_onReceive_http_request: http header is too long.
```
### POST file upload

最大サイズは [package_max_length](/server/setting?id=package_max_length) 設定に制限されています。デフォルトは2Mですが、[Server->set](/server/methods?id=set) を呼び出して新しい値を渡すことでサイズを変更できます。Swooleの基礎はすべてメモリ上で処理されるため、大きすぎる値を設定すると多くの並行リクエストによりサーバーリソースが枯渇する可能性があります。

計算方法：`Maximum memory usage` = `Maximum concurrent requests` * `package_max_length`
