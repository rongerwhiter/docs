# コルーチンプロセス管理

コルーチンスペース内で`fork`プロセスを行うと他のコルーチンコンテキストを持ってしまうため、内部では`Process`モジュールの使用を禁止しています。代わりに以下の方法を使用できます。

* `System::exec()`または`Runtime Hook`+`shell_exec`を使用して外部プログラムを実行する
* `Runtime Hook`+`proc_open`を使用して親子プロセス間の相互通信を実現する

## 使用例

### main.php

```php
use Swoole\Runtime;
use function Swoole\Coroutine\run;

Runtime::enableCoroutine(SWOOLE_HOOK_ALL);
run(function () {
    $descriptorspec = array(
        0 => array("pipe", "r"),
        1 => array("pipe", "w"),
        2 => array("file", "/tmp/error-output.txt", "a")
    );

    $process = proc_open('php ' . __DIR__ . '/read_stdin.php', $descriptorspec, $pipes);

    $n = 10;
    while ($n--) {
        fwrite($pipes[0], "hello #$n \n");
        echo fread($pipes[1], 8192);
    }

    fclose($pipes[0]);
    proc_close($process);
});
```

### read_stdin.php

```php
while(true) {
    $line = fgets(STDIN);
    if ($line) {
        echo $line;
    } else {
        break;
    }
}
```
