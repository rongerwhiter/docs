# Swoole

?> `Swoole`는 `C++` 언어로 작성된 비동기 이벤트 기반 및 코루틴을 기반으로 하는 병렬 네트워크 통신 엔진으로, `PHP`에 [코루틴](/coroutine) 및 [고성능](/question/use?id=how-is-the-performance-of-swoole) 네트워크 프로그래밍 지원을 제공합니다. 다양한 통신 프로토콜의 네트워크 서버 및 클라이언트 모듈을 제공하여 `TCP/UDP 서비스`, `고성능 웹`, `WebSocket 서비스`, `사물인터넷`, `실시간 통신`, `게임`, `마이크로 서비스` 등을 쉽고 빠르게 구현할 수 있습니다. 이를 통해 `PHP`는 더 이상 전통적인 웹 영역에만 국한되지 않게 됩니다.

## Swoole 클래스 다이어그램

!> 해당 문서 페이지로 직접 이동할 수 있습니다.

[//]: # (https://naotu.baidu.com/file/bd9d2ba7dfae326e6976f0c53f88b18c)

<embed src="/_images/swoole_class.svg" type="image/svg+xml" alt="Swoole 아키텍처 다이어그램" />

## 공식 웹사이트

* [Swoole 공식 웹사이트](//www.swoole.com)
* [상용 제품 및 지원](//business.swoole.com)
* [Swoole 질문과 답변](//wenda.swoole.com)

## 프로젝트 주소

* [GitHub](//github.com/swoole/swoole-src) **(Support appreciated with a Star)**
* [码云](//gitee.com/swoole/swoole)
* [PECL](//pecl.php.net/package/swoole)

## 개발 도구

* [IDE Helper](https://github.com/swoole/ide-helper)
* [Yasd](https://github.com/swoole/yasd)
* [debugger](https://github.com/swoole/debugger)

## 저작권 정보

본 문서의 원본 콘텐츠는 이전 [구버전 Swoole 문서](https://wiki.swoole.com/wiki/index/prid-1)로부터 발췌되었으며, 계속된 문서 문제에 대한 사용자들의 불평을 해소하고자 현대적인 문서 구성 형식을 채택하여 `Swoole4`의 내용만을 포함하고 있습니다. 오래된 문서의 오류를 많이 수정하고 문서 세부 사항을 최적화하며, 예제 코드와 교육 콘텐츠를 추가하여 `Swoole` 초보자에게 더욱 친숙하게 만들었습니다.

본 문서의 모든 내용, 텍스트, 이미지 및 오디오 비주얼 자료는 **상해 시월 네트워크 과학기술 유한회사**에게 권한이 있습니다. 어떤 미디어, 웹사이트 또는 개인도 외부 링크 형태로 인용할 수 있지만, 프로토콜 동의 없이 어떠한 형태로도 복사하여 발표/게시할 수는 없습니다.

## 문서 시작자

* 杨才 [GitHub](https://github.com/TTSimple)
* 郭新华 [Weibo](https://www.weibo.com/u/2661945152)
* [鲁飞](https://github.com/sy-records) [Weibo](https://weibo.com/5384435686)

## 피드백

본 문서의 내용 문제(오탈자, 예제 오류, 내용 부재 등) 및 요구 사항 제안은 모두 [swoole-inc/report](https://github.com/swoole-inc/report) 프로젝트에 통합하여 `issue`를 제출해 주세요. 또는 오른쪽 상단의 [피드백](/?id=main)을 클릭하여 직접 `issue` 페이지로 이동할 수도 있습니다.

수렴된 과거에 제안된 내용은 [문서 기여자](/CONTRIBUTING) 목록에 제출자 정보가 추가될 것이며 감사의 의미로 표시할 것입니다.

## 문서 원칙

명쾌한 언어 사용, `Swoole`의 기술적 세부사항과 하부 개념을 **가급적** 적게 소개하며, 추후 하부 부분은 별도의 `hack` 섹션을 유지해야 합니다;

일부 개념을 피할 수 없을 때는 반드시 해당 개념을 집중적으로 소개하는 중요한 곳이 있어야 하며, 다른 곳에서 내부 링크로 연결해야 합니다. 예: [이벤트 루프](/learn?id=what-is-eventloop) ;

문서를 작성할 때는 다른 사람들이 이해할 수 있는지를 미루어보면서 초보자의 관점에서 사고를 변화시켜야 합니다;

후속 기능 변경이 발생할 때는 꼭 모든 관련된 곳을 수정해야 하며, 한 곳만 수정하는 것이 아니어야 합니다;

각 기능 모듈은 반드시 완전한 예제를 가지고 있어야 합니다;
