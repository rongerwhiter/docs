# 関数リスト

Swooleは、ネットワーク通信に関連する関数以外にも、PHPプログラムがシステム情報を取得するためのいくつかの関数を提供しています。
## swoole_set_process_name()

プロセス名を設定するために使用されます。プロセス名を変更すると、`php your_file.php` ではなく指定した文字列が `ps` コマンドで表示されます。

この関数は1つの文字列引数を受け入れます。

この関数はPHP5.5で提供された[cli_set_process_title](https://www.php.net/manual/zh/function.cli-set-process-title.php)機能と同じです。ただし、`swoole_set_process_name` はPHP5.2以降の任意のバージョンで使用できます。`swoole_set_process_name` は `cli_set_process_title` よりも互換性が低く、`cli_set_process_title` 関数が存在する場合は、`cli_set_process_title` を優先して使用することをお勧めします。

```php
function swoole_set_process_name(string $name): void
```

使用例：

```php
swoole_set_process_name("swoole server");
```
### 各プロセスの名前を変更する方法

- [onStart](/server/events?id=onstart)が呼び出されるときに、メインプロセスの名前を変更する
- [onManagerStart](/server/events?id=onmanagerstart)が呼び出されるときに、マネージャプロセス(`manager`)の名前を変更する
- [onWorkerStart](/server/events?id=onworkerstart)が呼び出されるときに、workerプロセスの名前を変更する
 
!> 低いバージョンのLinuxカーネルとMac OSXではプロセスの名前変更がサポートされていません
## swoole_strerror()

エラーコードをエラーメッセージに変換します。

関数のプロトタイプ：

```php
function swoole_strerror(int $errno, int $error_type = 1): string
```

エラータイプ:

* `1`：標準の`Unix Errno`、システムコールのエラー、`EAGAIN`や`ETIMEDOUT`などが該当
* `2`：`getaddrinfo`のエラーコード、`DNS`操作によるもの
* `9`：`Swoole`の内部エラーコード、`swoole_last_error()`で得られるもの

使用例：

```php
var_dump(swoole_strerror(swoole_last_error(), 9));
```
## swoole_version()

Swoole拡張機能のバージョンを取得します。例： `1.6.10`

```php
function swoole_version(): string
```

使用例：

```php
var_dump(SWOOLE_VERSION); //グローバル変数SWOOLE_VERSIONもSwoole拡張機能のバージョンを表します
var_dump(swoole_version());
/**
戻り値：
string(6) "1.9.23"
string(6) "1.9.23"
**/
```
## swoole_errno()

最新のシステムコールのエラーコードを取得します。これは`C/C++`の`errno`変数と同等です。

```php
function swoole_errno(): int
```

エラーコードの値はオペレーティングシステムに依存しています。エラーメッセージに変換するには、`swoole_strerror`を使用できます。
## swoole_get_local_ip()

この関数は、すべてのネットワークインターフェースのIPアドレスを取得するために使用されます。

```php
function swoole_get_local_ip(): array
```

使用例：

```php
// すべてのネットワークインターフェースのIPアドレスを取得
$list = swoole_get_local_ip();
print_r($list);
/**
Return Value
Array
(
      [eno1] => 10.10.28.228
      [br-1e72ecd47449] => 172.20.0.1
      [docker0] => 172.17.0.1
)
**/
```

!>注意
* 現時点ではIPv4アドレスのみを返します。結果からローカルループアドレス127.0.0.1は除外されます。
* 結果の配列はインターフェース名をキーとした関連配列です。例えば `array("eth0" => "192.168.1.100")`
* この関数は`ioctl`システムコールをリアルタイムで使用してインターフェース情報を取得しますが、キャッシュはありません。
## swoole_clear_dns_cache()

`swoole_client`および`swoole_async_dns_lookup`に対して有効な、Swoole内部のDNSキャッシュをクリアします。

```php
function swoole_clear_dns_cache()
```
## swoole_get_local_mac()

本機能は、ローカルネットワークアダプタのMacアドレスを取得します。

```php
function swoole_get_local_mac(): array
```

* 成功した場合、すべてのネットワークアダプタのMacアドレスが返されます。

```php
array(4) {
  ["lo"]=>
  string(17) "00:00:00:00:00:00"
  ["eno1"]=>
  string(17) "64:00:6A:65:51:32"
  ["docker0"]=>
  string(17) "02:42:21:9B:12:05"
  ["vboxnet0"]=>
  string(17) "0A:00:27:00:00:00"
}
```
## swoole_cpu_num()

本機のCPUコア数を取得します。

```php
function swoole_cpu_num(): int
```

* 成功するとCPUコア数が返されます。例：

```shell
php -r "echo swoole_cpu_num();"
```
## swoole_last_error()

Swooleの最後のエラーコードを取得します。

```php
function swoole_last_error(): int
```

`swoole_strerror(swoole_last_error(), 9)`を使用してエラーをエラーメッセージに変換できます。エラーメッセージの完全なリストについては[Swooleエラーコードリスト](/other/errno?id=swoole)を参照してください。
```php
function swoole_mime_type_add(string $suffix, string $mime_type): bool
```
```php
function swoole_mime_type_set(string $suffix, string $mime_type): bool
``` 

`swoole_mime_type_set()`関数は、特定のMIMEタイプを変更します。存在しない場合は`false`が返されます。

## swoole_mime_type_delete()

特定のmimeタイプを削除します。存在しない場合は`false`を返します。

```php
function swoole_mime_type_delete(string $suffix): bool
```
```php
function swoole_mime_type_get(string $filename): string
````

`filename`に対応するMIMEタイプを取得します。
```php
function swoole_mime_type_exists(string $suffix): bool
````

関数`swolle_mime_type_exists()`は、指定された拡張子に対応するMIMEタイプが存在するかどうかを取得します。
## swoole_substr_json_decode()

零拷贝 JSON デシリアライズ。 `$offset` と `$length` 以外は、他の引数は [json_decode](https://www.php.net/manual/en/function.json-decode.php) と同じです。

!> Swoole バージョン >= `v4.5.6` で利用可能であり、`v4.5.7` 以降では、コンパイル時に[--enable-swoole-json](/environment?id=通用参数) パラメータを追加して有効にする必要があります。使用シナリオは[Swoole 4.5.6 サポート零拷贝 JSON または PHP デシリアライズ](https://wenda.swoole.com/detail/107587)を参照してください。

```php
function swoole_substr_json_decode(string $packet, int $offset, int $length, bool $assoc = false, int $depth = 512, int $options = 0)
```

  * **例**

```php
$val = json_encode(['hello' => 'swoole']);
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(json_decode(substr($str, 4, $l), true));
var_dump(swoole_substr_json_decode($str, 4, $l, true));
```
## swoole_substr_unserialize()

零コピーPHP逆シリアライズ、`$offset`および`$length`を除くすべてのパラメータは、他の `unserialize` と同じです。

!> Swoole バージョン >= `v4.5.6` で利用可能です。使用シナリオは[Swoole 4.5.6 支持零拷贝 JSON 或 PHP 反序列化](https://wenda.swoole.com/detail/107587)を参照してください

```php
function swoole_substr_unserialize(string $packet, int $offset, int $length, array $options= [])
```

  * **例**

```php
$val = serialize('hello');
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(unserialize(substr($str, 4, $l)));
var_dump(swoole_substr_unserialize($str, 4, $l));
```
## swoole_error_log()

エラーメッセージをログに出力します。`$level`は[ログレベル](/consts?id=ログレベル)です。

!> Swoole バージョン >= `v4.5.8` で利用可能

```php
function swoole_error_log(int $level, string $msg)
```
## swoole_clear_error()

清除ソケットのエラーまたは最後のエラーコード上のエラー。

!> Swoole バージョン >= `v4.6.0` で使用可能

```php
function swoole_clear_error()
```
## swoole_coroutine_socketpair()

[socket_create_pair](https://www.php.net/manual/en/function.socket-create-pair.php) のコルーチンバージョン。

!> Swoole バージョン >= `v4.6.0` で利用可能

```php
function swoole_coroutine_socketpair(int $domain , int $type , int $protocol): array|bool
```
## swoole_async_set

この関数は非同期`IO`に関連するオプションを設定できます。

```php
function swoole_async_set(array $settings)
```

- enable_signalfd `signalfd`機能の使用を有効または無効にします
- enable_coroutine 組み込みのコルーチンの有効化または無効化、[詳細はこちら](/server/setting?id=enable_coroutine)
- aio_core_worker_num AIOの最小プロセス数を設定します
- aio_worker_num AIOの最大プロセス数を設定します
## swoole_error_log_ex()

指定されたレベルとエラーコードでログを書き込みます。

```php
function swoole_error_log_ex(int $level, int $error, string $msg)
```

!> Swoole version >= `v4.8.1` is available.
## swoole_ignore_error()

指定されたエラーコードのエラーログを無視します。

```php
function swoole_ignore_error(int $error)
```

!> Swoole version >= `v4.8.1` available
