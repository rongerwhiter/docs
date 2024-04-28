# ini 설정

설정 | 기본값 | 용도
---|---|---
swoole.enable_coroutine | On | 내장 코루틴을 켜거나 끄는 옵션으로, 자세한 내용은 [여기](/server/setting?id=enable_coroutine)를 참조하세요.
swoole.display_errors | On | `Swoole` 오류 메시지를 표시하거나 숨기는 옵션입니다.
swoole.unixsock_buffer_size | 8M | 프로세스 간 통신에 사용되는 `Socket` 버퍼 크기를 설정하며, [socket_buffer_size](/server/setting?id=socket_buffer_size)와 동일합니다.
swoole.use_shortname | On | 짧은 별칭을 사용할지 여부를 설정하는 옵션으로, 자세한 내용은 [여기](/other/alias?id=协程短名称)를 참조하세요.
swoole.enable_preemptive_scheduler | Off | 일부 코루틴이 CPU 시간을 오랫동안 차지하여 다른 코루틴이 [스케줄링](/coroutine?id=协程调度)되지 못할 때를 방지하는데 도움이 되는 옵션으로, [예시](https://github.com/swoole/swoole-src/tree/master/tests/swoole_coroutine_scheduler/preemptive)를 참조하세요.
swoole.enable_library | On | 내장된 library를 활성화하거나 비활성화하는 옵션입니다.
