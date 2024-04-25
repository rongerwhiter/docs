# 서버 측(비동기 스타일)

간편하게 `TCP`, `UDP`, [unixSocket](/learn?id=What is IPC) 3 종류의 소켓 유형을 지원하는 비동기 서버 프로그램을 만들 수 있습니다. 또한 `IPv4` 및 `IPv6`를 지원하며, `SSL/TLS` 단방향 및 양방향 인증을 지원하는 터널 암호화 기능을 제공합니다. 사용자는 하위 구현 세부사항에 신경 쓸 필요가 없으며, 네트워크 [이벤트](/server/events)의 콜백 함수만 설정하면 됩니다. 예시는 [빠른 시작](/start/start_tcp_server)을 참고하세요.

!> `Server` 측의 스타일은 비동기입니다(즉, 모든 이벤트에 대해 콜백 함수를 설정해야 함)만, 동시에 코루틴도 지원합니다. [enable_coroutine](/server/setting?id=enable_coroutine)을 활성화하면(기본적으로 활성화) 코루틴을 사용할 수 있습니다. [코루틴](/coroutine) 아래의 모든 비즈니스 코드는 동기적으로 작성됩니다.

방문하여 더 알아보기:

[Server의 두 가지 실행 모드 소개](/learn?id=server의 두 가지 실행 모드 소개 ':target=_blank')  
[Process, ProcessPool, UserProcess의 차이점은 무엇인가요](/learn?id=process-diff ':target=_blank')  
[Master 프로세스, Reactor 스레드, Worker 프로세스, Task 프로세스, Manager 프로세스의 차이 및 관계](/learn?id=diff-process ':target=_blank')  

### 실행 흐름도 <!-- {docsify-ignore} --> 

![running_process](https://wiki.swoole.com/_images/server/running_process.png ':size=800xauto')

### 프로세스/스레드 구조도 <!-- {docsify-ignore} --> 

![process_structure](https://wiki.swoole.com/_images/server/process_structure.png ':size=800xauto')

![process_structure_2](https://wiki.swoole.com/_images/server/process_structure_2.png)
