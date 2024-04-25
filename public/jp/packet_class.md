# Swoole\Server\Packet

これは`Swoole\Server\Packet`の詳細です。

## Properties

### $server_socket
サーバーのファイルディスクリプタ`fd`を返します。この属性は整数型`int`です。

```php
Swoole\Server\Packet->server_socket
```

### $server_port
サーバーがリッスンしているポート`server_port`を返します。この属性は整数型`int`です。

```php
Swoole\Server\Packet->server_port
```

### $dispatch_time
リクエストデータが到着した時間`dispatch_time`を返します。この属性は浮動小数点型`double`です。

```php
Swoole\Server\Packet->dispatch_time
```

### $address
クライアントのアドレス`address`を返します。この属性は文字列型`string`です。

```php
Swoole\Server\Packet->address
```

### $port
クライアントがリッスンしているポート`port`を返します。この属性は整数型`int`です。

```php
Swoole\Server\Packet->port
```

### $data
クライアントが送信したデータ`data`を返します。この属性は文字列型`string`です。

```php
Swoole\Server\Packet->data
```
