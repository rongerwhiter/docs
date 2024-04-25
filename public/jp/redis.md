# 協力Redisクライアント

!> このクライアントの使用は推奨されていません。代わりに `Swoole\Runtime::enableCoroutine + phpredis` や `predis` を使用し、ネイティブの`PHP`の`redis`クライアントを[一括協力化](/runtime)してください。
## 使用例

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $val = $redis->get('key');
});
```

!> `subscribe` `pSubscribe`は`defer(true)`の場合には使用できません。
## 方法

!> 方法的使用基本与 [phpredis](https://github.com/phpredis/phpredis) 保持一致。

以下说明不同于[phpredis](https://github.com/phpredis/phpredis)的实现：

1、尚未実装されているRedisコマンド：`scan object sort migrate hscan sscan zscan`；

2、`subscribe pSubscribe`の使用方法では、コールバック関数の設定は不要です；

3、PHP変数のシリアル化のサポートは、`connect()`メソッドの第三引数を`true`に設定すると有効になります。デフォルトは`false`です。
```php
Swoole\Coroutine\Redis::__construct(array $options = null);
```  
### setOptions()

4.2.10バージョン以降、`Redis`クライアントの設定を構築および接続後に設定するための新しいメソッドが追加されました。

この関数はSwooleスタイルであり、`Key-Value`のキーと値のペア配列を使用して構成する必要があります。

```php
Swoole\Coroutine\Redis->setOptions(array $options): void
```

  * **設定可能なオプション**

key | 説明
---|---
`connect_timeout` | 接続タイムアウト、デフォルトはグローバルのコルーチン`socket_connect_timeout`(1秒)
`timeout` | タイムアウト、デフォルトはグローバルのコルーチン`socket_timeout`、参考[クライアントのタイムアウト規則](/coroutine_client/init?id=超時规则)
`serialize` | 自動シリアライズ、デフォルトはオフ
`reconnect` | 自動再接続試行回数、接続がタイムアウトなどの理由で`close`された場合、次回のリクエスト時に自動的に接続を試みてからリクエストを送信します。デフォルトは`1`回(`true`)で、指定回数失敗した後には再試行を行いません。接続のアクティブ維持にのみ使用され、リクエストの再送信により幂等でないインターフェースのエラーが発生することはありません。
`compatibility_mode` | `hmGet/hGetAll/zRange/zRevRange/zRangeByScore/zRevRangeByScore` 関数の戻り値が`php-redis`と一致しない場合の互換性解決策。有効にすると、`Co\Redis` および `php-redis` が同じ結果を返します。デフォルトはオフです。【この設定項目は`v4.4.0`以降で使用可能】
### set()

データを格納します。

```php
Swoole\Coroutine\Redis->set(string $key, mixed $value, array|int $option): bool
```

  * **Parameters** 

    * **`string $key`**
      * **機能**：データのキー
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $value`**
      * **機能**：データの内容【文字列以外の値は自動的にシリアル化されます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $options`**
      * **機能**：オプション
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `$option`の説明：  
      `整数`：有効期限を設定します。例：`3600`  
      `配列`：高度な有効期限の設定。例：`['nx', 'ex' => 10]` 、`['xx', 'px' => 1000]`

      !> `px`: ミリ秒単位の有効期限を表します。  
      `ex`: 秒単位の有効期限を表します。  
      `nx`: 存在しない場合にタイムアウトを設定します。  
      `xx`: 存在する場合にタイムアウトを設定します。
### request()

Redisサーバーにカスタムコマンドを送信します。これは、phpredisのrawCommandに類似しています。

```php
Swoole\Coroutine\Redis->request(array $args): void
```

  * **パラメータ** 

    * **`array $args`**
      * **機能**：配列形式のパラメータリスト。【最初の要素は`Redis`コマンドでなければならず、他の要素はコマンドのパラメータで、内部で`Redis`プロトコルリクエストに自動的にパッケージ化されて送信されます。】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

`Redis`サーバーがコマンドを処理する方法に応じて、数字、ブール値、文字列、配列などのタイプが戻ってくる可能性があります。

  * **使用例** 

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379); //もしローカルのUNIXソケットなら、hostパラメータは`unix://tmp/your_file.sock`のように入力する必要があります
    $res = $redis->request(['object', 'encoding', 'key1']);
    var_dump($res);
});
```
Sure! Here's the translation:

## Attributes

属性
### errCode

エラーコード。

エラーコード | 説明
---|---
1 | 読み取りまたは書き込みエラー
2 | その他すべて...
3 | ファイルの終わり
4 | プロトコルエラー
5 | メモリ不足
### errMsg

エラーメッセージ。
### 接続済み

現在の `Redis` クライアントがサーバーに接続しているかどうかを判定します。
## 定数

`multi($mode)`メソッドで使用されるデフォルトは`SWOOLE_REDIS_MODE_MULTI`モードです：

* SWOOLE_REDIS_MODE_MULTI
* SWOOLE_REDIS_MODE_PIPELINE

`type()`コマンドの戻り値を判断する際に使用される定数：

* SWOOLE_REDIS_TYPE_NOT_FOUND
* SWOOLE_REDIS_TYPE_STRING
* SWOOLE_REDIS_TYPE_SET
* SWOOLE_REDIS_TYPE_LIST
* SWOOLE_REDIS_TYPE_ZSET
* SWOOLE_REDIS_TYPE_HASH
## トランザクションモード

`Redis`のトランザクションモードは、`multi`と`exec`を使用して実装することができます。

  * **ヒント**

    * `mutli`命令を使用してトランザクションを開始し、その後のすべての命令がキューに追加されて実行を待ちます
    * `exec`命令を使用してトランザクション内のすべての操作を実行し、すべての結果を一度に取得します

  * **使用例**

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $redis->multi();
    $redis->set('key3', 'rango');
    $redis->get('key1');
    $redis->get('key2');
    $redis->get('key3');

    $result = $redis->exec();
    var_dump($result);
});
```
## サブスクリプションモード

!> Swooleのバージョンがv4.2.13以上で利用可能です。**4.2.12およびそれ以下のバージョンではサブスクリプションモードにバグがあります**
### Subscribe

`subscribe/psubscribe`は、`phpredis`とは異なり、コルーチンスタイルです。

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    if ($redis->subscribe(['channel1', 'channel2', 'channel3'])) // またはpsubscribeを使用することもできます
    {
        while ($msg = $redis->recv()) {
            // msgは配列で、以下の情報が含まれています
            // $type # 戻り値のタイプ：サブスクライブ成功を示す
            // $name # サブスクライブしたチャンネル名またはソースのチャンネル名
            // $info  # 現在サブスクライブしているチャンネルの数または情報コンテンツ
            list($type, $name, $info) = $msg;
            if ($type == 'subscribe') { // またはpsubscribe
                // チャンネルが正常にサブスクライブされたメッセージ、サブスクライブした数だけ受信
            } else if ($type == 'unsubscribe' && $info == 0){ // またはpunsubscribe
                break; // サブスクライブを解除するメッセージを受信し、残りのサブスクライブされたチャンネル数が0の場合、これ以上受信を停止してループを終了
            } else if ($type == 'message') {  // psubscribeの場合はここがpmessageになります
                var_dump($name); // ソースのチャンネル名を表示
                var_dump($info); // メッセージを表示する
                // balabalaba.... // メッセージを処理
                if ($need_unsubscribe) { // 特定の状況で解除が必要な場合
                    $redis->unsubscribe(); // 解除が完了するのを待ち続けるためにrecvを続ける
                }
            }
        }
    }
});
```
### Unsubscribe

Unsubscribe using `unsubscribe/punsubscribe`, `$redis->unsubscribe(['channel1'])`

At this point, `$redis->recv()` will receive an unsubscribe message. If you unsubscribe from multiple channels, you will receive multiple messages.

!> Note: After unsubscribing, make sure to continue calling `recv()` until you receive the last unsubscribe message (`$msg[2] == 0`). Once you receive this message, you can exit the subscription mode.

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    if ($redis->subscribe(['channel1', 'channel2', 'channel3'])) // or use psubscribe
    {
        while ($msg = $redis->recv()) {
            // msg is an array containing the following information
            // $type # return type: show subscription success
            // $name # subscribed channel name or source channel name
            // $info  # the number of channels or information content currently subscribed
            list($type, $name, $info) = $msg;
            if ($type == 'subscribe') // or psubscribe
            {
                // channel subscription success message
            }
            else if ($type == 'unsubscribe' && $info == 0) // or punsubscribe
            {
                break; // received the unsubscribe message, and the number of channels remaining for the subscription is 0, no longer received, break the loop
            }
            else if ($type == 'message') // if it's psubscribe，here is pmessage
            {
                // print source channel name
                var_dump($name);
                // print message
                var_dump($info);
                // handle messsage
                if ($need_unsubscribe) // in some cases, you need to unsubscribe
                {
                    $redis->unsubscribe(); // continue recv to wait unsubscribe finished
                }
            }
        }
    }
});
```
## 互換モード

`Co\Redis`の `hmGet/hGetAll/zrange/zrevrange/zrangebyscore/zrevrangebyscore`コマンドの結果が`phpredis`拡張の戻り値形式と一致しない問題は、解決されました [#2529](https://github.com/swoole/swoole-src/pull/2529)。

古いバージョンとの互換性を保つために、`$redis->setOptions(['compatibility_mode' => true]);` を設定すると、`Co\Redis`と`phpredis`の戻り値が一致することが保証されます。

!> Swooleのバージョン >= `v4.4.0` が必要です

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->setOptions(['compatibility_mode' => true]);
    $redis->connect('127.0.0.1', 6379);

    $co_get_val = $redis->get('novalue');
    $co_zrank_val = $redis->zRank('novalue', 1);
    $co_hgetall_val = $redis->hGetAll('hkey');
    $co_hmget_val = $redis->hmGet('hkey', array(3, 5));
    $co_zrange_val = $redis->zRange('zkey', 0, 99, true);
    $co_zrevrange_val = $redis->zRevRange('zkey', 0, 99, true);
    $co_zrangebyscore_val = $redis->zRangeByScore('zkey', 0, 99, ['withscores' => true]);
    $co_zrevrangebyscore_val = $redis->zRevRangeByScore('zkey', 99, 0, ['withscores' => true]);
});
```
