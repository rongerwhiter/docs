# 오류 보고서 제출

## 필요 사항

Swoole 커널 버그를 발견한 경우 보고서를 제출하십시오. Swoole의 커널 개발자들은 문제가 존재함을 아직 모를지도 모릅니다. 보고서를 제출하지 않으면 버그가 발견되지 못할 수도 있고 수정되기가 어려울 수도 있습니다. 오류 보고서는 [GitHub의 이슈 페이지](https://github.com/swoole/swoole-src/issues) 에서 제출할 수 있습니다(`New issue` 버튼을 클릭하세요). 여기서 제출된 오류 보고서는 최우선으로 처리됩니다.

이메일 목록이나 개인 메일로 오류 보고서를 보내지 마십시오. GitHub의 이슈 페이지에서도 Swoole에 대한 요청 및 제안을 할 수 있습니다.

오류 보고서를 제출하기 전에 **오류 보고서를 제출하는 방법**을 읽어주시기 바랍니다.

## 새 이슈 생성

이슈를 만들 때 시스템이 다음 템플릿을 제공할 것입니다. 신중히 작성해주시기 바랍니다. 정보가 부족할 경우 이슈가 무시될 수 있습니다:

```markdown

Please answer these questions before submitting your issue. Thanks!
> 제출하기 전에 다음 질문에 답변해주세요. 감사합니다!
	
1. What did you do? If possible，provide a simple script for reproducing the error.
> 어떤 작업을 했나요? 가능하다면 오류 재현을 위한 간단한 스크립트를 제공해주세요.

2. What did you expect to see?
> 보고서를 제출하기 전에 감흥했던 바를 적어주세요.

3. What did you see instead?
> 대신 본 내용이 무엇인가요?

4. What version of Swoole are you using (`php --ri swoole`)?
> 사용 중인 Swoole 버전은 무엇인가요? (`php --ri swoole` 출력 복사)

5. What is your machine environment used (including the version of kernel & php & gcc)?
> 사용 중인 기계 환경은 무엇인가요 (커널 및 PHP 및 gcc 버전을 포함하여)?

```

여기서 가장 중요한 것은 **오류 재현을 위한 간단한 스크립트를 제공**해야 합니다. 그렇지 않으면 개발자가 오류 원인을 판단하는 데 도움이 되도록 다른 정보를 최대한 제공해야 합니다.

## 메모리 분석 (강력 권장)

Valgrind은 gdb보다 더 많은 메모리 문제를 발견할 수 있습니다. 다음 명령어를 사용하여 프로그램을 실행하고 버그를 유발할 때까지 실행하세요.

```shell
USE_ZEND_ALLOC=0 valgrind --log-file=/tmp/valgrind.log php your_file.php
```

* 프로그램에서 오류가 발생하면 `ctrl+c`를 눌러 종료하고 `/tmp/valgrind.log` 파일을 업로드하여 개발 팀이 버그를 파악하는 데 도움을 줍니다.

## 세그멘테이션 오류(코어 덤프)

더불어 특정 경우에는 디버그 도구를 사용하여 문제를 파악하는 데 도움을 줄 수 있습니다.

```shell
WARNING	swManager_check_exit_status: worker#1 abnormal exit, status=0, signal=11
```

Swoole 로그에 위와 같은 메시지(signal11)가 나타나면 `코어 덤프`가 발생한 것입니다. 문제가 발생한 위치를 확인하기 위해 디버그 도구를 사용해야 합니다.

> `gdb`를 사용하여 `swoole`을 추적하기 전에 더 많은 정보를 보존하려면 컴파일 시 `--enable-debug` 매개변수를 추가해야 합니다.

코어 덤프 파일 활성화
```shell
ulimit -c unlimited
```

버그를 유발하면 코어 덤프 파일이 프로그램 디렉토리, 시스템 루트 디렉토리 또는 `/cores` 디렉토리에 생성됩니다(시스템 구성에 따라 다름).

다음 명령을 입력하여 gdb를 사용하여 프로그램을 디버깅하세요.

```
gdb php core
gdb php /tmp/core.1234
```

그런 다음 `bt`를 입력하고 엔터를 누르면 문제가 발생한 호출 스택이 표시됩니다.
```
(gdb) bt
```

`f 숫자`를 입력하여 특정 호출 스택 프레임을 볼 수 있습니다.
```
(gdb) f 1
(gdb) f 0
```

위 정보를 모두 이슈에 제공해주시기 바랍니다.
