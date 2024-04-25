# Swoole\Server\Event

`Swoole\Server\Event`の詳細について説明します。

## プロパティ

### $reactor_id
所属する`Reactor`スレッドのIDを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\Event->reactor_id
```

### $fd
接続のファイルディスクリプタ`fd`を返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\Event->fd
```

### $dispatch_time
リクエストデータの到着時間`dispatch_time`を返します。このプロパティは`double`型です。このプロパティは`onReceive`イベントでのみ`0`以外の値を持ちます。

```php
Swoole\Server\Event->dispatch_time
```

### $data
クライアントが送信したデータ`data`を返します。このプロパティは`string`型の文字列です。このプロパティは`onReceive`イベントでのみ`null`以外の値を持ちます。

```php
Swoole\Server\Event->data
```
