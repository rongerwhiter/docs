`Swoole`의 대다수 기능은 `cli` 명령줄 환경에서만 사용할 수 있습니다. 먼저 `Linux Shell` 환경을 준비하세요. 코드를 작성하기 위해 `Vim`, `Emacs`, `PhpStorm` 또는 다른 편집기를 사용할 수 있고, 아래 명령어를 사용하여 프로그램을 실행할 수 있습니다.

```shell
php /path/to/your_file.php
```

`Swoole` 서버 프로그램을 성공적으로 실행한 후 코드에 `echo` 문이 없는 경우 화면에는 아무 출력이 나타나지 않지만 실제로는 하위 수준에서 네트워크 포트를 수신하고 클라이언트의 연결을 기다리고 있습니다. 해당 서버에 연결하여 테스트하는 데 사용 가능한 클라이언트 도구와 프로그램을 사용할 수 있습니다.

#### 프로세스 관리

기본적으로 `Swoole` 서비스를 시작한 후 시작한 창에서 `CTRL+C`를 누르면 서비스를 종료할 수 있지만 창이 종료되면 문제가 발생할 수 있습니다. 백그라운드에서 실행해야 하는 경우 자세한 내용은 [데몬화](/server/setting?id=daemonize)를 참조하세요.

!> 간단한 예제는 대부분 비동기 스타일의 프로그래밍 방식입니다. 코루틴 스타일로도 동일한 기능을 수행할 수 있습니다. 관련 내용은 [서버 (코루틴 스타일)](coroutine/server.md)을 참조하세요.

!> `Swoole`가 제공하는 대부분의 모듈은 `cli` 명령줄 인터페이스에서만 사용할 수 있습니다. 현재는 [동기 차단 클라이언트](/client)만 `PHP-FPM` 환경에서 사용할 수 있습니다.