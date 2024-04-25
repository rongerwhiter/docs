# Coroutine\MySQL

协程MySQL客户端。

!> 本客户端不再推荐使用，推荐使用 Swoole\Runtime::enableCoroutine + PDO或Mysqli 方式，即[一键协程化](/runtime)原生 PHP 的 MySQL 客户端。

!> 请勿同时使用`Swoole1.x`时代的异步回调写法和本协程MySQL客户端。
```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $swoole_mysql = new MySQL();
    $swoole_mysql->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'user',
        'password' => 'pass',
        'database' => 'test',
    ]);
    $res = $swoole_mysql->query('select sleep(1)');
    var_dump($res);
});
``` 
## defer特性

以下に示す[並列クライアント](/coroutine/multi_call)を参照してください。
## ストアドプロシージャ

`4.0.0` バージョン以降、`MySQL` ストアドプロシージャおよび複数の結果セットの取得がサポートされています。
## MySQL8.0

`Swoole-4.0.1`またはそれ以降のバージョンでは、`MySQL8`のすべてのセキュリティ機能をサポートしており、パスワードの設定を戻す必要なく、クライアントを正常に使用できます。
### 4.0.1 以下のバージョン

`MySQL-8.0`は安全性の高い`caching_sha2_password`プラグインがデフォルトで使用されます。`5.x`からアップグレードしてきた場合、すべての`MySQL`機能を直接使用できますが、新しく作成された`MySQL`の場合は、次の手順を実行する必要があります。

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
flush privileges;
```

ステートメント中の `'root'@'localhost'` を使用しているユーザーに置き換え、`password` をそのパスワードに置き換えます。

もしまだ使用できない場合は、my.cnf で `default_authentication_plugin = mysql_native_password` を設定する必要があります。
```plaintext
Name: Fire
Element: Heat
```

## 属性

```plaintext
名前：火
要素：熱
```
### serverInfo

The connection information that is stored is an array passed to the connection function.
### 靴下

接続に使用されるファイル記述子。
### connected

`MySQL`サーバーに接続されていますか。

!>参考：[connected 属性和连接状态不一致](/question/use?id=connected属性和连接状态不一致)
### connect_error

`connect`を実行する際のエラーメッセージ。
### connect_errno

`connect`関数を実行する際のエラーコード、整数型です。
### エラー

サーバーが`MySQL`の命令を実行しようとしたときに返されるエラーメッセージ。
### errno

`MySQL`コマンドを実行する際に、サーバーから返されるエラーコードで、整数型です。
### 影響を受けた行数

影響を受けた行数。
### insert_id

最後に挿入されたレコードの`id`。
## Methods
### connect()

MySQLの接続を確立します。

```php
Swoole\Coroutine\MySQL->connect(array $serverInfo): bool
```

!> `$serverInfo`：パラメータは配列の形で渡されます

```php
[
    'host'        => 'MySQLのIPアドレス', // ローカルUNIXソケットの場合、`unix://tmp/your_file.sock`のような形式で記入してください
    'user'        => 'データベースユーザー',
    'password'    => 'データベースのパスワード',
    'database'    => 'データベース名',
    'port'        => 'MySQLのポート、デフォルトは3306、オプション',
    'timeout'     => '接続タイムアウト時間', // connectのタイムアウト時間にのみ影響し、queryとexecuteメソッドには影響しない、クライアントのタイムアウトルールを参照してください
    'charset'     => '文字セット',
    'strict_type' => false, // ストリクトモードを有効にし、queryメソッドで返されるデータも強制的に型変換されます
    'fetch_mode'  => true,  // フェッチモードを有効にし、pdoと同じようにfetch/fetchAllを使用して1行ずつ結果を取得するか、結果セット全体を取得するか(バージョン4.0以上)
]
```
### query()

SQL文を実行します。

```php
Swoole\Coroutine\MySQL->query(string $sql, float $timeout = 0): array|false
```

  * **パラメータ** 

    * **`string $sql`**
      * **機能**：SQL文
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間 【指定された時間内に`MySQL`サーバーがデータを返さなかった場合、`false`を返し、エラーコードを`110`に設定して接続を切断します】
      * **単位**：秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`0`
      * **その他の値**：なし
      * **参考[クライアントのタイムアウト規則](/coroutine_client/init?id=超時規則)**

  * **戻り値**

    * タイムアウト/エラーの場合は`false`、それ以外は`array`形式でクエリ結果が返されます。

  * **遅延受信**

  !> `defer`を設定した後、`query`を呼び出すと直ちに`true`が返ります。`recv`を呼び出すと、協力的に待機してクエリの結果が返されます。

  * **例**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $swoole_mysql = new MySQL();
    $swoole_mysql->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'user',
        'password' => 'pass',
        'database' => 'test',
    ]);
    $res = $swoole_mysql->query('show tables');
    if ($res === false) {
        return;
    }
    var_dump($res);
});
```
### prepare()

MySQLサーバーにSQLプリペアドリクエストを送信します。

!> `prepare`は`execute`と一緒に使用する必要があります。プリペアドリクエストが成功した後、`execute`メソッドを呼び出して`MySQL`サーバーにデータパラメータを送信します。

```php
Swoole\Coroutine\MySQL->prepare(string $sql, float $timeout): Swoole\Coroutine\MySQL\Statement|false;
```

  * **Parameters** 

    * **`string $sql`**
      * **Purpose**: プリペアドステートメント【`?`をパラメータプレースホルダとして使用】
      * **デフォルト値**: なし
      * **その他の値**: なし

    * **`float $timeout`**
      * **Purpose**: タイムアウト時間 
      * **単位**: 秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**: `0`
      * **その他の値**: なし
      * **[クライアントのタイムアウトルール](/coroutine_client/init?id=超時規則)を参照**

  * **Return Value**

    * 失敗した場合は`false`が返ります。エラーの原因を調べるために`$db->error`と`$db->errno`をチェックできます。
    * 成功した場合は`Coroutine\MySQL\Statement`オブジェクトが返ります。オブジェクトの[execute](/coroutine_client/mysql?id=statement-gtexecute)メソッドを呼び出してパラメータを送信できます。

  * **Example**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $ret1 = $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $stmt = $db->prepare('SELECT * FROM userinfo WHERE id=?');
    if ($stmt == false) {
        var_dump($db->errno, $db->error);
    } else {
        $ret2 = $stmt->execute(array(10));
        var_dump($ret2);
    }
});
```
### escape()

SQL文の特殊文字をエスケープして、SQLインジェクション攻撃を防ぎます。`mysqlnd`ベースの関数を使用しており、`PHP`の`mysqlnd`拡張機能に依存しています。

!> コンパイル時に[--enable-mysqlnd](/environment?id=编译选项)を追加して有効にする必要があります。

```php
Swoole\Coroutine\MySQL->escape(string $str): string
```

  * **パラメーター** 

    * **`string $str`**
      * **機能**：エスケープする文字列
      * **デフォルト値**：なし
      * **他の値**：なし

  * **使用例**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $data = $db->escape("abc'efg\r\n");
});
```
### begin()

トランザクションを開始します。 `commit` および `rollback` と組み合わせて `MySQL` トランザクション処理を実装します。

```php
Swoole\Coroutine\MySQL->begin(): bool
```

!> `MySQL` トランザクションを開始し、成功した場合は `true` を返し、失敗した場合は `false` を返します。エラーコードを取得するには `$db->errno` を確認してください。

!> 同じ `MySQL` 接続オブジェクトでは、同時に 1 つのトランザクションしか開始できません；
前のトランザクションが `commit` または `rollback` されるまで新しいトランザクションを開始することができません；
それ以外の場合、基になる部分では `Swoole\MySQL\Exception` 例外がスローされ、例外の `code` は `21` になります。

  * **例**

    ```php
    $db->begin();
    $db->query("update userinfo set level = 22 where id = 1");
    $db->commit();
    ```
### commit()

トランザクションをコミットします。

!> `begin` と一緒に使用する必要があります。

```php
Swoole\Coroutine\MySQL->commit(): bool
```

!> 成功した場合は `true` を返し、失敗した場合は `false` を返します。 エラーコードを取得するには `$db->errno` をチェックしてください。
### rollback()

トランザクションをロールバックします。

!> 必ず`begin`と一緒に使用する必要があります。

```php
Swoole\Coroutine\MySQL->rollback(): bool
```

!> 成功した場合は`true`が返され、失敗した場合は`false`が返されます。エラーコードを取得するには`$db->errno`を確認してください。
### Statement->execute()

MySQLサーバーにSQLプリペアドデータパラメータを送信します。

!> `execute` must be used in conjunction with `prepare`, and `prepare` must be called before calling `execute` to initiate a prepared request.

!> The `execute` method can be called repeatedly.

```php
Swoole\Coroutine\MySQL\Statement->execute(array $params, float $timeout = -1): array|bool
```

  * **Parameters** 

    * **`array $params`**
      * **Description**: Prepared data parameters【Must be the same as the number of parameters in the `prepare` statement. `$params` must be an array with numeric indexes, and the order of the parameters must be the same as the `prepare` statement】
      * **Default**: None
      * **Other values**: None

    * **`float $timeout`**
      * **Description**: Timeout period【If the MySQL server does not return data within the specified time, it will return `false`, set the error code to `110`, and disconnect the connection】
      * **Unit**: seconds, with a minimum precision of milliseconds (`0.001` seconds)
      * **Default**: `-1`
      * **Other values**: None
      * **Refer to [Client Timeout Rules](/coroutine_client/init?id=超时规则)**

  * **Return Value**

    * Returns `true` on success, or if the `fetch_mode` parameter of `connect` is set to `true`
    * Returns an array of data sets on success if not in the above case,
    * Returns `false` on failure, and you can check `$db->error` and `$db->errno` to determine the cause of the error

  * **Usage Example** 

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $ret1 = $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $stmt = $db->prepare('SELECT * FROM userinfo WHERE id=? and name=?');
    if ($stmt == false) {
        var_dump($db->errno, $db->error);
    } else {
        $ret2 = $stmt->execute(array(10, 'rango'));
        var_dump($ret2);

        $ret3 = $stmt->execute(array(13, 'alvin'));
        var_dump($ret3);
    }
});
```
### Statement->fetch()

結果セットから次の行を取得します。

```php
Swoole\Coroutine\MySQL\Statement->fetch(): ?array
```

!> Swooleバージョン >= `4.0-rc1` では、`connect`時に`fetch_mode => true`オプションを指定する必要があります。

  * **例** 

```php
$stmt = $db->prepare('SELECT * FROM ckl LIMIT 1');
$stmt->execute();
while ($ret = $stmt->fetch()) {
    var_dump($ret);
}
```

!> 新しい`MySQL`ドライバー（`v4.4.0`以降）では、`fetch`を`NULL`まで読み取るのに例の方法で使用する必要があります。そうしないと新しいリクエストを発行できなくなります（必要に応じた読み取り機構により、メモリを節約できます）。
### Statement->fetchAll()

結果セット内のすべての行を含む配列を返します。

```php
Swoole\Coroutine\MySQL\Statement->fetchAll():? array
```

!> Swoole versions >= `4.0-rc1` require `fetch_mode => true` option in `connect`

  * **Example**

```php
$stmt = $db->prepare('SELECT * FROM ckl LIMIT 1');
$stmt->execute();
$stmt->fetchAll();
```
### Statement->nextResult()

In a multi-response result statement handle, proceed to the next response result (such as multiple results returned by stored procedures).

```php
Swoole\Coroutine\MySQL\Statement->nextResult():? bool
```

  * **Return Value**

    * Returns `TRUE` on success
    * Returns `FALSE` on failure
    * Returns `NULL` if there is no next result

  * **Example** 

    * **Non-fetch mode**

    ```php
    $stmt = $db->prepare('CALL reply(?)');
    $res  = $stmt->execute(['hello mysql!']);
    do {
      var_dump($res);
    } while ($res = $stmt->nextResult());
    var_dump($stmt->affected_rows);
    ```

    * **Fetch mode**

    ```php
    $stmt = $db->prepare('CALL reply(?)');
    $stmt->execute(['hello mysql!']);
    do {
      $res = $stmt->fetchAll();
      var_dump($res);
    } while ($stmt->nextResult());
    var_dump($stmt->affected_rows);
    ```

!> Starting from the new `MySQL` driver of `v4.4.0`, in the `fetch` mode, one must read until getting `NULL`  as shown in the example code, otherwise new requests cannot be initiated (due to the underlying on-demand reading mechanism, which can save memory).
