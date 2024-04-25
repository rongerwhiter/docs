# 协程编程须知

使用 Swoole [协程](/coroutine) 特性，请认真阅读本章节编程须知。
## プログラミングパラダイム

- コルーチン内ではグローバル変数の使用は禁止されています
- コルーチンは`use`キーワードを使用して外部変数を現在のスコープに持ち込む際に、参照の使用は禁止されています
- コルーチン間の通信には[Channel](/coroutine/channel)を必ず使用してください

!> つまり、コルーチン間の通信にはグローバル変数を使用したり、外部変数を現在のスコープに持ち込むことはせず、必ず`Channel`を使用してください

- プロジェクトで`zend_execute_ex`または`zend_execute_internal`をフックする拡張機能がある場合は、Cスタックに特に注意する必要があります。[Co::set](/coroutine/coroutine?id=set)を使用してCスタックのサイズを再設定できます

!> これらのエントリーポイントをフックすると、一般的にPHP命令の呼び出しがC関数の呼び出しに変わり、Cスタックの消費が増加することがあります。
## コルーチンの終了

Swooleの古いバージョンでは、コルーチン内で`exit`を使用してスクリプトを強制終了するとメモリエラーが発生し、予期しない結果や`coredump`が発生する可能性があります。Swooleサーバーで`exit`を使用すると、サービスプロセス全体が終了し、内部のすべてのコルーチンが異常終了するため、深刻な問題が発生します。Swooleは長い間、開発者が`exit`を使用することを禁止していますが、開発者は例外をスローして、非標準の方法で、トップレベルの`catch`で`exit`と同様の終了ロジックを実装することができます。

!> バージョン4.2.2以上では、スクリプト（`http_server`が作成されていない場合）は現在のコルーチンのみが存在する状況で`exit`を実行することができます。

Swoole **v4.1.0** 以降は、`コルーチン`や`サービスのイベントループ`内でPHPの`exit`を直接サポートしています。この場合、内部的にはキャッチ可能な`Swoole\ExitException`が自動的にスローされ、開発者は必要な位置でキャッチして、オリジナルのPHPと同じような終了ロジックを実装することができます。
### Swoole\ExitException

`Swoole\ExitException`は`Exception`クラスを継承し、`getStatus`と`getFlags`という2つのメソッドが追加されています。

```php
namespace Swoole;

class ExitException extends \Exception
{
    public function getStatus(): mixed
    public function getFlags(): int
}
```
```php
public function getStatus(): mixed
````

`getStatus()`メソッドは、`exit($status)`が呼び出されたときに渡された`status`パラメータを取得します。このメソッドは、任意の変数型をサポートしています。
#### getFlags()

exitが発生したときの環境情報のマスクを取得します。

```php
public function getFlags(): int
```

現在、次のマスクがあります：

| 定数 | 説明 |
| -- | -- |
| SWOOLE_EXIT_IN_COROUTINE | コルーチン中に終了 |
| SWOOLE_EXIT_IN_SERVER | サーバー内で終了 |
### Instructions
```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

function route()
{
    controller();
}

function controller()
{
    your_code();
}

function your_code()
{
    Coroutine::sleep(.001);
    exit(1);
}

run(function () {
    try {
        route();
    } catch (\Swoole\ExitException $e) {
        var_dump($e->getMessage());
        var_dump($e->getStatus() === 1);
        var_dump($e->getFlags() === SWOOLE_EXIT_IN_COROUTINE);
    }
});
```
```php
use function Swoole\Coroutine\run;

$exit_status = 0;
run(function () {
    try {
        exit(123);
    } catch (\Swoole\ExitException $e) {
        global $exit_status;
        $exit_status = $e->getStatus();
    }
});
var_dump($exit_status);
```
## 例外処理

Coroutineプログラミングでは、`try/catch`を使用して例外を処理することができます。**ただし、例外をキャッチする場合は必ずコルーチン内で行う必要があり、コルーチンをまたいで例外をキャッチすることはできません**。

!> `Exception`がスローされているアプリケーションレベルだけでなく、一部のエラーが捕捉できることに注意が必要です。これには、`function`、`class`、`method`が存在しない場合などが該当します。
### エラーの例

以下のコードでは、`try/catch`と`throw`が異なるコルーチンで行われているため、コルーチン内ではこの例外をキャッチできません。コルーチンが終了すると、キャッチされていない例外があり、致命的なエラーが発生します。

```bash
PHP Fatal error:  Uncaught RuntimeException
```

```php
try {
	Swoole\Coroutine::create(function () {
		throw new \RuntimeException(__FILE__, __LINE__);
	});
}
catch (\Throwable $e) {
	echo $e;
}
```
### 正しい例

例外をコルーチン内でキャッチする。

```php
function test() {
	throw new \RuntimeException(__FILE__, __LINE__);
}

Swoole\Coroutine::create(function () {
	try {
		test();
	}
	catch (\Throwable $e) {
		echo $e;
	}
});
```
## `__get` / `__set` マジックメソッド内でコルーチン切り替えを生成しないでください

理由：[PHP7内核剖析](https://github.com/pangudashu/php7-internal/blob/40645cfe087b373c80738881911ae3b178818f11/3/zend_object.md)

> **Note:** もしクラスに`__get()`メソッドが存在する場合、オブジェクトのプロパティメモリ（つまり：properties_table）を割り当てる際に、さらにHashTableというタイプのzvalが追加で割り当てられます。これはいわゆる循環呼び出しを防ぐために行われ、例を挙げると：
> 
> ***public function __get($var) { return $this->$var; }***
>
> このような状況では、`__get()`を呼び出す際に存在しないプロパティにアクセスしてしまい、`__get()`メソッド内で再帰的に呼び出すことになります。もしリクエストされた$varが判断されずにそのまま再帰呼び出しが続けられると、無限に再帰することになります。そのため、`__get()`を呼び出す前に現在の$varが既に`__get()`内に存在するかどうかを確認し、もしそうであれば再度`__get()`を呼び出しません。そうでなければ、$varをキーとしてそのHashTableに挿入し、ハッシュ値を次のように設定します：*guard |= IN_ISSET、`__get()`が終了した後にハッシュ値を次のように設定します：*guard &= ~IN_ISSET。
>
> このHashTableは`__get()`だけでなく、他のマジックメソッドでも使用されます。そのため、そのハッシュ値の型はzend_longであり、異なるマジックメソッドが異なるビット位置を占めています。さらに、全てのオブジェクトがこのHashTableを追加で割り当てるわけではありません。オブジェクトの作成時に ***zend_class_entry.ce_flags*** が ***ZEND_ACC_USE_GUARDS*** を含んでいるかどうかに基づいて、HashTableを割り当てる必要性が決定されます。オブジェクトが作成される際に、`__get()`、`__set()`、`__unset()`、`__isset()`メソッドが定義されている場合、ce_flagsにこのマスクが付けられます。

コルーチン切り替えを行った後、次回呼び出し時に再帰呼び出しと判断され、この問題はPHPの**特性**によるものであり、PHP開発チームとの問題解決についての対処は現時点で行われていません。

注意：マジックメソッド内にはコルーチン切り替えを生成するコードがなくても、プリエンプティブなスケジューリングが有効になっている場合、マジックメソッドが強制的に別のコルーチンに切り替わる可能性があります。

お勧め：`get`/`set`メソッドを自分で実装して明示的に呼び出すこと

オリジナル問題リンク：[#2625](https://github.com/swoole/swoole-src/issues/2625)
## Serious Error

The following actions will result in a serious error.
### 複数のコルーチンで1つの接続を共有する

同期ブロッキングプログラムとは異なり、コルーチンはリクエストを並行して処理するため、同時に複数のリクエストが処理されることがあります。クライアント接続を共有すると、異なるコルーチン間でデータが混在する可能性があります。参照：[複数のコルーチンでTCP接続を共有する](/question/use?id=client-has-already-been-bound-to-another-coroutine)
```python
# クラスの静的変数/グローバル変数を使用してコンテキストを保存する

複数のコルーチンは同時に実行されるため、コルーチンのコンテキスト情報を保存するためにはクラスの静的変数やグローバル変数を使用できません。ローカル変数を使用するのは安全です。ローカル変数の値は自動的にコルーチンのスタックに保存されるため、他のコルーチンからはアクセスできません。
```
#### サンプルエラー

```php
$server = new Swoole\Http\Server('127.0.0.1', 9501);

$_array = [];
$server->on('request', function ($request, $response) {
    global $_array;
    // /a をリクエスト（コルーチン1）
    if ($request->server['request_uri'] == '/a') {
        $_array['name'] = 'a';
        co::sleep(1.0);
        echo $_array['name'];
        $response->end($_array['name']);
    }
    // /b をリクエスト（コルーチン2）
    else {
        $_array['name'] = 'b';
        $response->end();
    }
});
$server->start();
```

`2` つの並行リクエストを発行します。

```shell
curl http://127.0.0.1:9501/a
curl http://127.0.0.1:9501/b
```

- コルーチン1 でグローバル変数 `$_array['name']` の値を `a` に設定
- コルーチン1 が `co::sleep` を実行して一時停止
- コルーチン2 が実行され、`$_array['name']` の値を `b` に設定してコルーチン2 終了
- この時、タイマーがリターンされ、バックグラウンドでコルーチン1 が再開。しかし、コルーチン1 のロジックにはコンテキストの依存関係があります。`$_array['name']` の値を再度出力すると、プログラムの期待値は `a` であるべきですが、この値は既にコルーチン2 によって変更されており、実際の結果は `b` になってしまい、これによりロジックエラーが発生します。
- 同様に、クラスの静的変数 `Class::$array`、グローバルオブジェクト属性 `$object->array`、他のスーパーグローバル変数 `$GLOBALS` などを使用して、コルーチンプログラム内にコンテキストを保存するのは非常に危険です。予期しない動作が発生する可能性があります。

![](../_images/coroutine/notice-1.png)
#### 正しい例：Contextを使用してコンテキストを管理する

コルーチンのコンテキストを管理するために、`Context`クラスを使用できます。`Context`クラスでは、`Coroutine::getuid`を使用してコルーチンIDを取得し、異なるコルーチン間でグローバル変数を隔離し、コルーチンが終了するとコンテキストデータをクリーンアップします。

```php
use Swoole\Coroutine;

class Context
{
    protected static $pool = [];

    static function get($key)
    {
        $cid = Coroutine::getuid();
        if ($cid < 0)
        {
            return null;
        }
        if(isset(self::$pool[$cid][$key])){
            return self::$pool[$cid][$key];
        }
        return null;
    }

    static function put($key, $item)
    {
        $cid = Coroutine::getuid();
        if ($cid > 0)
        {
            self::$pool[$cid][$key] = $item;
        }
    }

    static function delete($key = null)
    {
        $cid = Coroutine::getuid();
        if ($cid > 0)
        {
            if($key){
                unset(self::$pool[$cid][$key]);
            }else{
                unset(self::$pool[$cid]);
            }
        }
    }
}
```

使用例：

```php
use Swoole\Coroutine\Context;

$server = new Swoole\Http\Server('127.0.0.1', 9501);

$server->on('request', function ($request, $response) {
    if ($request->server['request_uri'] == '/a') {
        Context::put('name', 'a');
        co::sleep(1.0);
        echo Context::get('name');
        $response->end(Context::get('name'));
        //コルーチン終了時にクリーンアップ
        Context::delete('name');
    } else {
        Context::put('name', 'b');
        $response->end();
        //コルーチン終了時にクリーンアップ
        Context::delete();
    }
});
$server->start();
```
