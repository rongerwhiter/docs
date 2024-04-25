# Linux 시그널 목록
## 완전한 대조표

| 신호      | 값       | 기본 동작 | 의미(신호를 발생시킨 이유)            |
| --------- | -------- | -------- | ----------------------------------- |
| SIGHUP    | 1        | Term     | 터미널 연결 끊김 또는 프로세스 사망  |
| SIGINT    | 2        | Term     | 키보드에서의 인터럽트 신호            |
| SIGQUIT   | 3        | Core     | 키보드에서의 종료 신호                |
| SIGILL    | 4        | Core     | 부정한 명령                         |
| SIGABRT   | 6        | Core     | abort에서의 예외 신호                |
| SIGFPE    | 8        | Core     | 부동 소수점 예외                    |
| SIGKILL   | 9        | Term     | 강제 종료                           |
| SIGSEGV   | 11       | Core     | 세그멘트 위반 오류(유효하지 않은 메모리 참조) |
| SIGPIPE   | 13       | Term     | 파이프 손상: 읽는 프로세스가 없는 파이프에 데이터를 쓰렀을 때 |
| SIGALRM   | 14       | Term     | 알람 타이머 시그널                  |
| SIGTERM   | 15       | Term     | 종료                                |
| SIGUSR1   | 30,10,16 | Term     | 사용자 정의 신호 1                  |
| SIGUSR2   | 31,12,17 | Term     | 사용자 정의 신호 2                  |
| SIGCHLD   | 20,17,18 | Ign      | 자식 프로세스가 중지 또는 종료      |
| SIGCONT   | 19,18,25 | Cont     | 일시 중지된 경우 계속 실행          |
| SIGSTOP   | 17,19,23 | Stop     | 터미널 외부에서의 중지 신호          |
| SIGTSTP   | 18,20,24 | Stop     | 터미널에서의 중지 신호               |
| SIGTTIN   | 21,21,26 | Stop     | 백그라운드 프로세스가 터미널에서 읽기 시도 |
| SIGTTOU   | 22,22,27 | Stop     | 백그라운드 프로세스가 터미널에 쓰기 시도 |
|           |          |          |                                   |
| SIGBUS    | 10,7,10  | Core     | 버스 오류(메모리 접근 오류)         |
| SIGPOLL   |          | Term     | 폴러블 이벤트 발생(Sys V), SIGIO와 동의어 |
| SIGPROF   | 27,27,29 | Term     | 프로파일 타이머가 만료됨          |
| SIGSYS    | 12,-,12  | Core     | 부적절한 시스템 호출(SVr4)          |
| SIGTRAP   | 5        | Core     | 추적/트랩 자가                    |
| SIGURG    | 16,23,21 | Ign      | 소켓 긴급 신호(4.2BSD)             |
| SIGVTALRM | 26,26,28 | Term     | 가상 타이머 만료(4.2BSD)           |
| SIGXCPU   | 24,24,30 | Core     | CPU 시간 제한 초과(4.2BSD)         |
| SIGXFSZ   | 25,25,31 | Core     | 파일 크기 제한 초과(4.2BSD)        |
|           |          |          |                                   |
| SIGIOT    | 6        | Core     | IOT 자가, SIGABRT와 동의어          |
| SIGEMT    | 7,-,7    |          | Term                              |
| SIGSTKFLT | -,16,-   | Term     | 부동 소수점 처리기 스택 오류(사용 안 함) |
| SIGIO     | 23,29,22 | Term     | 디스크립터에서 I/O 작업 가능        |
| SIGCLD    | -,-,18   | Ign      | SIGCHLD와 동의어                   |
| SIGPWR    | 29,30,19 | Term     | 전원 장애(System V)                |
| SIGINFO   | 29,-,-   |          | SIGPWR와 동의어                    |
| SIGLOST   | -,-,-    | Term     | 파일 잠금 손실                      |
| SIGWINCH  | 28,28,20 | Ign      | 창 크기 변경(4.3BSD, Sun)          |
| SIGUNUSED | -,31,-   | Term     | 사용되지 않는 신호(SIGSYS가 될 것임) |
## 신뢰할 수 없는 신호

| 이름      | 설명                         |
| --------- | ---------------------------- |
| SIGHUP    | 연결 끊김                    |
| SIGINT    | 터미널 인터럽트               |
| SIGQUIT   | 터미널 종료                   |
| SIGILL    | 부적절한 하드웨어 명령어      |
| SIGTRAP   | 하드웨어 오류                 |
| SIGABRT   | 비정상 종료(abort)            |
| SIGBUS    | 하드웨어 오류                 |
| SIGFPE    | 산술 예외                    |
| SIGKILL   | 종료                         |
| SIGUSR1   | 사용자 정의 신호              |
| SIGUSR2   | 사용자 정의 신호              |
| SIGSEGV   | 잘못된 메모리 참조            |
| SIGPIPE   | 읽기 없는 파이프에 쓰기      |
| SIGALRM   | 타이머 만료(alarm)            |
| SIGTERM   | 종료                         |
| SIGCHLD   | 자식 프로세스 상태 변경       |
| SIGCONT   | 일시 중지된 프로세스 계속     |
| SIGSTOP   | 중지                         |
| SIGTSTP   | 터미널 중지                   |
| SIGTTIN   | 백그라운드에서 터미널로 읽기  |
| SIGTTOU   | 백그라운드에서 터미널로 쓰기  |
| SIGURG    | 긴급 상황(소켓)              |
| SIGXCPU   | CPU 한도 초과(setrlimit)      |
| SIGXFSZ   | 파일 길이 한도 초과(setrlimit)|
| SIGVTALRM | 가상 시간 알람(setitimer)     |
| SIGPROF   | 프로파일 타임 아웃(setitimer)|
| SIGWINCH  | 터미널 창 크기 변경          |
| SIGIO     | 비동기 I/O                  |
| SIGPWR    | 전원 장애/재시작             |
| SIGSYS    | 잘못된 시스템 호출           |
## 신뢰할 수 있는 신호

| 이름        | 사용자 정의 |
| ----------- | ---------- |
| SIGRTMIN    |            |
| SIGRTMIN+1  |            |
| SIGRTMIN+2  |            |
| SIGRTMIN+3  |            |
| SIGRTMIN+4  |            |
| SIGRTMIN+5  |            |
| SIGRTMIN+6  |            |
| SIGRTMIN+7  |            |
| SIGRTMIN+8  |            |
| SIGRTMIN+9  |            |
| SIGRTMIN+10 |            |
| SIGRTMIN+11 |            |
| SIGRTMIN+12 |            |
| SIGRTMIN+13 |            |
| SIGRTMIN+14 |            |
| SIGRTMIN+15 |            |
| SIGRTMAX-14 |            |
| SIGRTMAX-13 |            |
| SIGRTMAX-12 |            |
| SIGRTMAX-11 |            |
| SIGRTMAX-10 |            |
| SIGRTMAX-9  |            |
| SIGRTMAX-8  |            |
| SIGRTMAX-7  |            |
| SIGRTMAX-6  |            |
| SIGRTMAX-5  |            |
| SIGRTMAX-4  |            |
| SIGRTMAX-3  |            |
| SIGRTMAX-2  |            |
| SIGRTMAX-1  |            |
| SIGRTMAX    |            |
