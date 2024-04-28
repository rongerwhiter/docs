# Coroutine\PostgreSQL 旧版

コルーチン `PostgreSQL` クライアント。この機能を有効化するには [ext-postgresql](https://github.com/swoole/ext-postgresql) 拡張をコンパイルする必要があります。

> このドキュメントは Swoole < 5.0 にのみ適用されます。
## コンパイルとインストール

ソースコードをダウンロードします：[https://github.com/swoole/ext-postgresql](https://github.com/swoole/ext-postgresql)。必ず、Swoole のリリースバージョンと対応する必要があります。

* システムに`libpq`ライブラリがインストールされていることを確認してください
* `mac`では`postgresql`をインストールすると`libpq`ライブラリが含まれていますが、環境によって異なります。`ubuntu`では`apt-get install libpq-dev`が必要な場合があり、`centos`では`yum install postgresql10-devel`が必要な場合があります
* `libpq`ライブラリのディレクトリを明示的に指定することもできます。例：`./configure --with-libpq-dir=/etc/postgresql`
## 使用例

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
    if (!$conn) {
        var_dump($pg->error);
        return;
    }
    $result = $pg->query('SELECT * FROM test;');
    $arr = $pg->fetchAll($result);
    var_dump($arr);
});
```
### 事务处理

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
    $pg->query('BEGIN');
    $result = $pg->query('SELECT * FROM test');
    $arr = $pg->fetchAll($result);
    $pg->query('COMMIT');
    var_dump($arr);
});
```
## Properties
### エラー

エラーメッセージを取得します。
## 方法

This text does not require translation.
### connect()

`postgresql`の非同期接続を確立します。

```php
Swoole\Coroutine\PostgreSQL->connect(string $connection_string): bool
```

!> `$connection_string`は接続情報であり、接続に成功するとtrueを返し、失敗するとfalseを返します。エラー情報は[error](/coroutine_client/postgresql?id=error)プロパティを使用して取得できます。
  * **例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=");
    var_dump($pg->error, $conn);
});
```
### query()

SQLステートメントを実行します。非同期非ブロッキングのコルーチンコマンドを送信します。

```php
Swoole\Coroutine\PostgreSQL->query(string $sql): resource;
```

  * **Parameters** 

    * **`string $sql`**
      * **機能**：SQLステートメント
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **Examples**

    * **select**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
        $result = $pg->query('SELECT * FROM test;');
        $arr = $pg->fetchAll($result);
        var_dump($arr);
    });
    ```

    * **insert idを返す**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=");
        $result = $pg->query("insert into test (id,text) VALUES (24,'text') RETURNING id ;");
        $arr = $pg->fetchRow($result);
        var_dump($arr);
    });
    ```

    * **transaction**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
        $pg->query('BEGIN;');
        $result = $pg->query('SELECT * FROM test;');
        $arr = $pg->fetchAll($result);
        $pg->query('COMMIT;');
        var_dump($arr);
    });
    ```
### fetchAll()

```php
Swoole\Coroutine\PostgreSQL->fetchAll(resource $queryResult, $resultType = SW_PGSQL_ASSOC):? array;
```

  * **パラメータ**
    * **`$resultType`**
      * **説明**：定数。オプションパラメータで、返り値の初期化方法を制御します。
      * **デフォルト値**：`SW_PGSQL_ASSOC`
      * **他の値**：なし

      値 | 返り値
      ---|---
      SW_PGSQL_ASSOC | フィールド名をキーとした連想配列を返す
      SW_PGSQL_NUM | フィールド番号をキーとした配列を返す
      SW_PGSQL_BOTH | 両方をキーとした配列を返す

  * **返り値**

    * 結果からすべての行を取得して配列として返します。
### affectedRows()

受影響したレコード数を返します。

```php
Swoole\Coroutine\PostgreSQL->affectedRows(resource $queryResult): int
```
### numRows()

```php
Swoole\Coroutine\PostgreSQL->numRows(resource $queryResult): int
```
### fetchObject()

一行をオブジェクトとして取得します。

```php
Swoole\Coroutine\PostgreSQL->fetchObject(resource $queryResult, int $row): object;
```

  * **Example**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    for ($row = 0; $row < $pg->numRows($result); $row++) {
        $data = $pg->fetchObject($result, $row);
        echo $data->id . " \n ";
    }
});
```
```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    while ($data = $pg->fetchObject($result, $row)) {
        echo $data->id . " \n ";
        $row++;
    }
});
```
### fetchAssoc()

一行を関連配列として取得します。

```php
Swoole\Coroutine\PostgreSQL->fetchAssoc(resource $queryResult, int $row): array
```
### fetchArray()

一行を配列として取得します。

```php
Swoole\Coroutine\PostgreSQL->fetchArray(resource $queryResult, int $row, $resultType = SW_PGSQL_BOTH): array|false
```

  * **パラメータ**
    * **`int $row`**
      * **役割**：`row` は取得したい行（レコード）の番号です。最初の行は `0` です。
      * **デフォルト値**：なし
      * **その他の値**：なし
    * **`$resultType`**
      * **役割**：定数です。オプションパラメータで、返り値の初期化方法を制御します。
      * **デフォルト値**：`SW_PGSQL_BOTH`
      * **その他の値**：なし

      値 | 返り値
      ---|---
      SW_PGSQL_ASSOC | フィールド名をキーとした連想配列を返します
      SW_PGSQL_NUM | フィールド番号をキーとして返します
      SW_PGSQL_BOTH | 両方をキーとして返します

  * **返り値**

    * 取得した行（タプル/レコード）に対応する配列を返します。取得可能な行がない場合は `false` を返します。

  * **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    $arr = $pg->fetchArray($result, 1, SW_PGSQL_ASSOC);
    var_dump($arr);
});
```
### fetchRow()

`result` リソースに基づいて、1行のデータ（レコード）を配列として取得して返します。各列が配列に順番に格納され、オフセット `0` から始まります。

```php
Swoole\Coroutine\PostgreSQL->fetchRow(resource $queryResult, int $row, $resultType = SW_PGSQL_NUM): array|false
```

  * **パラメータ**
    * **`int $row`**
      * **説明**：取得したい行（レコード）の番号。最初の行は `0` です。
      * **デフォルト値**：なし
      * **その他の値**：なし
    * **`$resultType`**
      * **説明**：定数。オプションのパラメータで、返り値の初期化方法を制御します。
      * **デフォルト値**：`SW_PGSQL_NUM`
      * **その他の値**：なし

      値 | 返り値
      ---|---
      SW_PGSQL_ASSOC | キーがフィールド名となる連想配列を返す
      SW_PGSQL_NUM | キーがフィールド番号となる配列を返す
      SW_PGSQL_BOTH | 両方をキーとして使用して返す

  * **返り値**

    * 返される配列は取得した行と同じ形式です。もし取得可能な行 `row` がない場合は `false` を返します。

  * **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    while ($row = $pg->fetchRow($result)) {
        echo "name: $row[0]  mobile: $row[1]" . PHP_EOL;
    }
});
```
### metaData()

表のメタデータを表示します。非同期非ブロッキングコルーチン版。

```php
Swoole\Coroutine\PostgreSQL->metaData(string $tableName): array
```
    
  * **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->metaData('test');
    var_dump($result);
});
```
### prepare()

准备处理。

```php
Swoole\Coroutine\PostgreSQL->prepare(string $name, string $sql);
Swoole\Coroutine\PostgreSQL->execute(string $name, array $bind);
```

  * **使用示例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=112");
    $pg->prepare("my_query", "select * from  test where id > $1 and id < $2");
    $res = $pg->execute("my_query", array(1, 3));
    $arr = $pg->fetchAll($res);
    var_dump($arr);
});
```
