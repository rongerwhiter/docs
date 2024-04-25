# Swoole\Server\TaskResult

`Swoole\Server\TaskResult`에 대한 자세한 설명이 여기 있습니다.

## 속성

### $task_id
`Reactor` 스레드 ID를 반환하며 이 속성은 정수형 `int`입니다.

```php
Swoole\Server\TaskResult->task_id
```

### $task_worker_id
이 실행 결과가 어느 `task` 프로세스에서 왔는지를 반환하며, 이 속성은 정수형 `int`입니다.

```php
Swoole\Server\TaskResult->task_worker_id
```

### $dispatch_time
연결된 데이터 `data`를 반환하며, 이 속성은 문자열형 `?string`입니다.

```php
Swoole\Server\TaskResult->dispatch_time
```

### $data
연결된 데이터 `data`를 반환하며, 이 속성은 문자열형 `string`입니다.

```php
Swoole\Server\StatusInfo->data
```
