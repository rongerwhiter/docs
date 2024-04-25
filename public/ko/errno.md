# 오류 코드

현재 오류 코드를 얻으려면 `swoole_last_error()`를 사용하실 수 있습니다;

`Swoole`의 하위 오류 코드를 텍스트 오류 정보로 변환하려면 `swoole_strerror(int $errno, 9);`을 사용하실 수 있습니다;

```php
echo swoole_strerror(swoole_last_error(), 9) . PHP_EOL;
echo swoole_strerror(SWOOLE_ERROR_MALLOC_FAIL, 9) . PHP_EOL;
```
