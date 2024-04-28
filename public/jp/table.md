# 高性能共有メモリーテーブル

`PHP`言語がマルチスレッドをサポートしていないため、`Swoole`は多プロセスモードを使用し、多プロセスモードではプロセス間メモリ分離があり、ワーカープロセス内で`global`グローバル変数とスーパーグローバル変数を修正しても他のプロセスには影響しません。

> `worker_num=1`に設定すると、プロセスの分離がないため、グローバル変数を使用してデータを保存できます

```php
$fds = array();
$server->on('connect', function ($server, $fd){
    echo "connection open: {$fd}\n";
    global $fds;
    $fds[] = $fd;
    var_dump($fds);
});
```

`$fds`はグローバル変数ですが、現在のプロセス内でのみ有効です。`Swoole`サーバーの内部では複数の`Worker`プロセスが作成されますが、`var_dump($fds)`で印刷される値には一部の接続`fd`のみが含まれます。

対応策は外部記憶サービスを使用することです：

* データベース、例：`MySQL`、`MongoDB`
* キャッシュサーバー、例：`Redis`、`Memcache`
* ディスクファイル、複数プロセス同時読み書き時にはロックが必要です

通常のデータベースやディスクファイルの操作には多くの`IO`待機時間があります。したがって、次のようなものを推奨します：

* `Redis` メモリーデータベース、読み書き速度が非常に速いですが、TCP接続などの問題があり、パフォーマンスが最高ではありません。
* `/dev/shm` メモリーファイルシステム、すべての読み書き操作はメモリ内で行われ、`IO`コストがなく、非常に高いパフォーマンスですが、データはフォーマットされず、データ同期の問題があります。

?> これらの記憶を使用する以外に、データを保存するために共有メモリを使用することをお勧めします。`Swoole\Table`は、超高性能な共有メモリとロックに基づいて実装された同時データ構造です。多プロセス/多スレッドデータの共有や同期ロックの問題を解決するために使用されます。`Table`のメモリ容量は`PHP`の`memory_limit`によって制御されません。

!> `Table`に対する配列方式の読み書きは避けてください。操作を行うには、必ずドキュメントで提供されているAPIを使用してください。  
`Table\Row`オブジェクトで配列方式を取り出すと、使い捨てのオブジェクトであり、多くの操作に依存しないでください。
バージョン `v4.7.0` 以降、配列方式での`Table`の読み書きをサポートしなくなり、`Table\Row`オブジェクトが削除されました。

* **利点**

  * 非常に高性能、単一スレッドで1秒あたり200万回の読み書きが可能；
  * アプリケーションコードにロックが不要、`Table`は行ロックでスピンロックが組み込まれており、すべての操作はマルチスレッド/マルチプロセスで安全です。ユーザーレベルではデータ同期の問題を考慮する必要はありません；
  * マルチプロセスをサポートし、`Table`は複数のプロセス間でデータを共有するために使用できます；
  * 行ロックを使用し、グローバルロックではなく、2つのプロセスが同じ`CPU`時間で同じデータを読み取るときにのみロックの競合が発生します。

* **繰り返し**

!> 繰り返し中に削除操作を行わないでください（すべての`key`を取得してから削除してください）

`Table`クラスはイテレータと`Countable`インターフェースを実装しており、`foreach`を使用して順次処理し、`count`を使用して現在の行数を計算できます。

```php
foreach($table as $row)
{
  var_dump($row);
}
echo count($table);
```
```json
{
    "name": "Alice",
    "age": 25,
    "sex": "female"
}
```
### size

テーブルの最大行数を取得します。

```php
Swoole\Table->size;
```
### memorySize

実際に使用されているメモリサイズをバイト単位で取得します。

```php
Swoole\Table->memorySize;
```
## Methods
### __construct()

内存テーブルを作成します。

```php
Swoole\Table::__construct(int $size, float $conflict_proportion = 0.2);
```

  * **引数** 

    * **`int $size`**
      * **機能**：テーブルの最大行数を指定します
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `Table`は共有メモリの上に構築されているため、動的に拡張できません。そのため、`$size`は作成前に自分で計算して設定する必要があります。`Table`が格納できる最大行数は`$size`と関連しますが、完全に一致するわけではありません。たとえば、`$size`が`1024`の場合、実際に格納できる行数は`1024`よりも**小さく**なります。もし`$size`が大きすぎて、マシンのメモリが不足している場合、`Table`の作成に失敗します。

    * **`float $conflict_proportion`**
      * **機能**：ハッシュ衝突の最大比率
      * **デフォルト値**：`0.2` (つまり`20%`)
      * **その他の値**：最小は`0.2`、最大は`1`

  * **容量の計算**

    * もし`$size`が`2`の`N`乗でない場合、例えば`1024`や`8192`、`65536`など、低位は自動的に最も近い数字に調整されます。`1024`よりも小さい場合はデフォルトで`1024`になります。`v4.4.6`バージョンから最小値は`64`になりました。
    * `Table`が使用する総メモリ量は (`HashTableの構造体の長さ` + `KEYの長さ64バイト` + `$sizeの値`) * (`1 + $conflict_proportionがハッシュ衝突率として使える`) * (`列のサイズ`)です。
    * もしあなたのデータの`Key`とハッシュ衝突率が`20%`を超える場合、衝突メモリブロックの容量が不足しているため、新しいデータを`set`しようとすると`Unable to allocate memory`エラーが発生し、`false`が返されて格納に失敗します。この場合は`$size`の値を増やしてサービスを再起動する必要があります。
    * 十分なメモリがある場合は、この値をできるだけ大きく設定してください。
```php
Swoole\Table->column(string $name, int $type, int $size = 0);
```

  * **パラメータ** 

    * **`string $name`**
      * **機能**：フィールドの名前を指定します
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $type`**
      * **機能**：フィールドのタイプを指定します
      * **デフォルト値**：なし
      * **その他の値**：`Table::TYPE_INT`, `Table::TYPE_FLOAT`, `Table::TYPE_STRING`

    * **`int $size`**
      * **機能**：文字列フィールドの最大長を指定します【文字列タイプのフィールドは`$size`を指定する必要があります】
      * **値の単位**：バイト
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **`$type` タイプの説明**

タイプ | 説明
---|---
Table::TYPE_INT | デフォルトは8バイトです
Table::TYPE_STRING | 設定後、指定した最大長（`$size`）を超える文字列は設定できません
Table::TYPE_FLOAT | 8バイトのメモリを占有します
```
### create()

メモリーテーブルを作成します。テーブルの構造を定義した後、`create`を実行してオペレーティングシステムにメモリーを要求し、テーブルを作成します。

```php
Swoole\Table->create(): bool
```

`create`メソッドを使用してテーブルを作成した後、実際に使用されるメモリーのサイズを取得するには[memorySize](/memory/table?id=memorysize)属性を使用できます。

  * **ヒント** 

    * `set`、`get`などのデータ読み取り／書き込み操作メソッドを使用する前に`create`を呼び出すことはできません
    * `create`を呼び出した後、`column`メソッドを使用して新しいフィールドを追加することはできません
    * システムのメモリーが不足しているため、要求が失敗した場合は`create`は`false`を返します
    * メモリーの要求が成功した場合は、`create`は`true`を返します

    !> `Table`はデータを保持するために共有メモリーを使用しています。子プロセスを作成する前に、`Table->create()`を必ず実行してください。  
    `Server`内で`Table`を使用する場合、`Table->create()`は`Server->start()`の前に実行する必要があります。

  * **使用例**

```php
$table = new Swoole\Table(1024);
$table->column('id', Swoole\Table::TYPE_INT);
$table->column('name', Swoole\Table::TYPE_STRING, 64);
$table->column('num', Swoole\Table::TYPE_FLOAT);
$table->create();

$worker = new Swoole\Process(function () {}, false, false);
$worker->start();

//$serv = new Swoole\Server('127.0.0.1', 9501);
//$serv->start();
```
### set()

行のデータを設定します。`Table`は`key-value`の方法でデータにアクセスします。

```php
Swoole\Table->set(string $key, array $value): bool
```

  * **パラメータ** 

    * **`string $key`**
      * **目的**：データの`key`
      * **デフォルト値**：なし
      * **他の値**：なし

      !> 同じ`$key`は同じ行のデータに対応し、`set`で同じ`key`を設定すると、前回のデータが上書きされます。`key`の最大長は63バイトを超えてはいけません。

    * **`array $value`**
      * **目的**：データの`value`
      * **デフォルト値**：なし
      * **他の値**：なし

      !> 配列でなければならず、フィールド定義の`$name`と完全に一致している必要があります。

  * **戻り値**

    * 成功した場合は`true`
    * 失敗した場合は`false`、ハッシュが多すぎて動的なスペースにメモリを割り当てることができない可能性があります。構築方法の二番目のパラメーターを大きくすることができます。

!> -`Table->set()` はすべてのフィールドの値を設定することも、一部のフィールドだけを変更することもできます。  
   -`Table->set()` を設定しない場合、その行のすべてのフィールドは空になります。  
   -`set`/`get`/`del` は行ロックが付いているので、`lock`を呼び出す必要はありません。  
   -**Key はバイナリセーフではなく、文字列型でなければなりません。バイナリデータを渡すことはできません。**

  * **使用例**

```php
$table->set('1', ['id' => 1, 'name' => 'test1', 'age' => 20]);
$table->set('2', ['id' => 2, 'name' => 'test2', 'age' => 21]);
$table->set('3', ['id' => 3, 'name' => 'test3', 'age' => 19]);
```

  * **最大サイズを超える文字列を設定する**

    文字列の長さが列の定義時に設定された最大サイズを超える場合、内部で自動的に切り取ります。

    ```php
    $table->column('str_value', Swoole\Table::TYPE_STRING, 5);
    $table->set('hello', array('str_value' => 'world 123456789'));
    var_dump($table->get('hello'));
    ```

    * `str_value`列の最大サイズは5バイトですが、5バイトを超える文字列が`set`で設定されます
    * 内部で5バイトのデータが自動的に切り取られ、最終的に`str_value`の値は`world`になります。

!> `v4.3`から、メモリ長さに対してアライメント処理が行われます。文字列の長さは8の倍数でなければならず、長さが5の場合、8バイトにアライメントされます。したがって、`str_value`の値は`world 12`になります。
### incr()

原子増加オペレーション。

```php
Swoole\Table->incr(string $key, string $column, mixed $incrby = 1): int
```

  * **パラメータ** 

    * **`string $key`**
      * **機能**：データの`key`【`$key`に対応する行が存在しない場合、デフォルトの列の値は`0`です】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $column`**
      * **機能**：指定された列名【浮動小数点数と整数フィールドのみサポートされます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $incrby`**
      * **機能**：増分【列が`int`の場合、`$incrby`は`int`型である必要があり、列が`float`型の場合は、`$incrby`は`float`型である必要があります】
      * **デフォルト値**：`1`
      * **その他の値**：なし

  * **戻り値**

    最終的な結果数値
### decr()

原子減算操作。

```php
Swoole\Table->decr(string $key, string $column, mixed $decrby = 1): int
```

  * **パラメータ** 

    * **`string $key`**
      * **機能**：データの`key`【`$key`に対応する行が存在しない場合、デフォルトで列の値は`0`になります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $column`**
      * **機能**：指定された列の名前【浮動小数点型と整数型のフィールドのみサポートされています】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $decrby`**
      * **機能**：増分 【列が`int`の場合、`$decrby`は`int`型である必要があり、列が`float`型の場合、`$decrby`は`float`型である必要があります】
      * **デフォルト値**：`1`
      * **その他の値**：なし

  * **戻り値**

    最終的な結果数値が返されます

    !> 数値が`0`の場合、減算操作により負の数になります
### get()

1行のデータを取得します。

```php
Swoole\Table->get(string $key, string $field = null): array|false
```

  * **Parameters** 

    * **`string $key`**
      * **Description**: データの`key`【文字列型である必要があります】
      * **Default**: なし
      * **Other values**: なし

    * **`string $field`**
      * **Description**: `$field`を指定すると、そのフィールドの値のみが返され、レコード全体ではありません
      * **Default**: なし
      * **Other values**: なし
      
  * **Return Value**

    * `$key`が存在しない場合、`false`を返す
    * 成功した場合、結果の配列を返す
    * `$field`が指定されている場合、そのフィールドの値のみが返され、レコード全体ではありません
### exist()

あるキーがテーブルに存在するかどうかを確認します。

```php
Swoole\Table->exist(string $key): bool
```

  * **パラメータ** 

    * **`string $key`**
      * **説明**：データの`key`【文字列型である必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし
```php
Swoole\Table->count(): int
```

テーブルに存在する項目数を返します。
### del()

データを削除します。

!> `Key`はバイナリセーフではありません。文字列型でなければならず、バイナリデータを渡してはいけません。**イテレーション中に削除しないでください**。

```php
Swoole\Table->del(string $key): bool
```

  * **戻り値**

    * `$key`に対応するデータが存在しない場合は`false`を返します。
    * 削除に成功した場合は`true`を返します。
### stats()

`Swoole\Table`の状態を取得します。

```php
Swoole\Table->stats(): array
```

!> Swooleバージョン>= `v4.8.0`で利用可能
## Helper function: id = swoole_table

1つの`Swoole\Table`を素早く作成するための関数。

```php
function swoole_table(int $size, string $fields): Swoole\Table
```

!> Swoole バージョン >= `v4.6.0` で利用可能です。`$fields` の形式は `foo:i/foo:s:num/foo:f` です。

| 短い名前 | 長い名前   | タイプ               |
| ---- | ------ | ------------------ |
| i    | int    | Table::TYPE_INT    |
| s    | string | Table::TYPE_STRING |
| f    | float  | Table::TYPE_FLOAT  |

例：

```php
$table = swoole_table(1024, 'fd:int, reactor_id:i, data:s:64');
var_dump($table);

$table = new Swoole\Table(1024, 0.25);
$table->column('fd', Swoole\Table::TYPE_INT);
$table->column('reactor_id', Swoole\Table::TYPE_INT);
$table->column('data', Swoole\Table::TYPE_STRING, 64);
$table->create();
var_dump($table);
```
```php
<?php
$table = new Swoole\Table(1024);
$table->column('fd', Swoole\Table::TYPE_INT);
$table->column('reactor_id', Swoole\Table::TYPE_INT);
$table->column('data', Swoole\Table::TYPE_STRING, 64);
$table->create();

$serv = new Swoole\Server('127.0.0.1', 9501);
$serv->set(['dispatch_mode' => 1]);
$serv->table = $table;

$serv->on('receive', function ($serv, $fd, $reactor_id, $data) {

	$cmd = explode(" ", trim($data));

	//get
	if ($cmd[0] == 'get')
	{
		//get self
		if (count($cmd) < 2)
		{
			$cmd[1] = $fd;
		}
		$get_fd = intval($cmd[1]);
		$info = $serv->table->get($get_fd);
		$serv->send($fd, var_export($info, true)."\n");
	}
	//set
	elseif ($cmd[0] == 'set')
	{
		$ret = $serv->table->set($fd, array('reactor_id' => $data, 'fd' => $fd, 'data' => $cmd[1]));
		if ($ret === false)
		{
			$serv->send($fd, "ERROR\n");
		}
		else
		{
			$serv->send($fd, "OK\n");
		}
	}
	else
	{
		$serv->send($fd, "command error.\n");
	}
});

$serv->start();
```  
