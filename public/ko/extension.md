# 충돌 확장

일부 추적 디버깅 `PHP` 확장은 전역 변수를 많이 사용하므로 `Swoole` 코루틴이 충돌할 수 있습니다. 관련 확장을 비활성화하세요:

* phptrace
* aop
* molten
* xhprof
* phalcon(`Swoole` 코루틴은 `phalcon` 프레임워크에서 실행될 수 없습니다)


## Xdebug 지원
`5.1` 버전부터 `xdebug` 확장을 직접 사용하여 `Swoole` 프로그램을 디버깅할 수 있습니다. 명령행 매개변수를 이용하거나 `php.ini`를 수정하여 다음을 활성화하세요.

```ini
swoole.enable_fiber_mock=On
```

또는

```shell
php -d swoole.enable_fiber_mock=On your_file.php
```
