# Swoole\Server\PipeMessage

`Swoole\Server\PipeMessage`の詳細についてこちらに説明があります。

## プロパティ

### $source_worker_id
データのソースワーカーIDを返します。このプロパティは整数型の`int`です。

```php
Swoole\Server\PipeMessage->source_worker_id
```

### $dispatch_time
リクエストデータが到着した時間`dispatch_time`を返します。このプロパティは浮動小数点数型の`double`です。

```php
Swoole\Server\PipeMessage->dispatch_time
```

### $data
接続されたデータを返します。このプロパティは文字列型の`string`です。

```php
Swoole\Server\PipeMessage->data
```
