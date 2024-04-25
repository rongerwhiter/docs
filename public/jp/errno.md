# エラーコード

現在のエラーコードは`swoole_last_error()`で取得できます；

`Swoole`のエラーコードをテキストエラー情報に変換するには、`swoole_strerror(int $errno, 9);`を使用できます；

```php
echo swoole_strerror(swoole_last_error(), 9) . PHP_EOL;
echo swoole_strerror(SWOOLE_ERROR_MALLOC_FAIL, 9) . PHP_EOL;
```
