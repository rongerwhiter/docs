# Swoole\Server\PipeMessage

이 문서는 `Swoole\Server\PipeMessage`에 대한 자세한 설명을 제공합니다.

## 속성

### $source_worker_id
데이터의 원본 `worker` 프로세스 ID를 반환합니다. 이 속성은 `int` 타입의 정수입니다.

```php
Swoole\Server\PipeMessage->source_worker_id
```

### $dispatch_time
요청 데이터가 도착한 시간인 `dispatch_time`을 반환합니다. 이 속성은 `double` 타입입니다.

```php
Swoole\Server\PipeMessage->dispatch_time
```

### $data
연결에 해당하는 데이터 `data`를 반환합니다. 이 속성은 `string` 타입의 문자열입니다.

```php
Swoole\Server\PipeMessage->data
```
