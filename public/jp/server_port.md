# Swoole\Server\Port

`Swoole\Server\Port`の詳細について説明します。

## プロパティ

### $host
リッスンするホストアドレスを返します。このプロパティは、`string`型の文字列です。

```php
Swoole\Server\Port->host
```

### $port
リッスンするホストのポート番号を返します。このプロパティは、`int`型の整数です。

```php
Swoole\Server\Port->port
```

### $type
このサーバーのタイプを返します。このプロパティは、`SWOOLE_TCP`、`SWOOLE_TCP6`、`SWOOLE_UDP`、`SWOOLE_UDP6`、`SWOOLE_UNIX_DGRAM`、`SWOOLE_UNIX_STREAM`のいずれかを返す列挙型です。

```php
Swoole\Server\Port->type
```

### $sock
リッスンしているソケットを返します。このプロパティは、`int`型の整数です。

```php
Swoole\Server\Port->sock
```

### $ssl
`ssl`暗号化が有効かどうかを返します。このプロパティは、`bool`型です。

```php
Swoole\Server\Port->ssl
```

### $setting
このポートの設定を返します。このプロパティは、`array`の配列です。

```php
Swoole\Server\Port->setting
```

### $connections
このポートに接続しているすべての接続を返します。このプロパティは、イテレータです。

```php
Swoole\Server\Port->connections
```

## メソッド

### set() 

`Swoole\Server\Port`の実行時の各種パラメータを設定するためのメソッドです。使用法は、[Swoole\Server->set()](/server/methods?id=set)と同様です。

```php
Swoole\Server\Port->set(array $setting): void
```

### on() 

`Swoole\Server\Port`のコールバック関数を設定するためのメソッドです。使用法は、[Swoole\Server->on()](/server/methods?id=on)と同様です。

```php
Swoole\Server\Port->on(string $event, callable $callback): bool
```

### getCallback() 

設定されたコールバック関数を返します。

```php
Swoole\Server\Port->getCallback(string $name): ?callback
```

  * **パラメータ**

    * `string $name`

      * 機能：コールバックイベントの名前
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * コールバック関数が存在する場合は成功、存在しない場合は`null`を返します。


### getSocket() 

現在のソケット`fd`をPHPの`Socket`オブジェクトに変換します。

```php
Swoole\Server\Port->getSocket(): Socket|false
```

  * **戻り値**

    * `Socket`オブジェクトが成功したことを示し、`false`が失敗したことを示します。

!> 注意：`--enable-sockets`を有効にして`Swoole`をコンパイルした場合にのみ、この関数を使用できます。
