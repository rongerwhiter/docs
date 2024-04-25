# 코루틴 프로세스 관리

코루틴 공간 내에서 `fork`로 프로세스를 만들 때 다른 코루틴 컨텍스트를 가져오므로 내부에서는 `Process` 모듈을 사용할 수 없습니다. 대신에 다음을 사용할 수 있습니다.

- `System::exec()` 또는 `Runtime Hook`+`shell_exec`를 사용하여 외부 프로그램 실행 구현
- `Runtime Hook`+`proc_open`을 사용하여 부모-자식 프로세스 상호작용 통신 구현

## 사용 예시

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
