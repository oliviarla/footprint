# item 72) 표준 예외를 사용하라

## 표준 예외

* 자바 플랫폼 라이브러리에는 대부분의 API가 필요로 하는 기본적인 exception들을 제공하고 있다.
* 표준 예외를 사용하면 대부분의 프로그래머에게 익숙해진 규약을 따르는 것이다.
* 다른사람이 사용하기 쉬운 API를 만들 수 있다.
* 가독성이 좋아진다.
* 예외 클래스 개수가 적을수록 메모리 사용량이 줄고 클래스를 적재하는 시간도 적게 걸린다.

## 재사용되는 예외 종류

* IllegalArgumentException: 호출자가 인수로 부적절한 값을 넘길 때
* IllegalStateException: 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때
* NullPointerException: null 값을 허용하지 않는 메서드에 null을 건낼 때
* IndexOutOfBoundsException: 어떠한 시퀀스의 허용 범위 넘을 때
* ConcurrentModificationException: 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려고 할 때
* UnsupportedOperationException: 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때
* 더 많은 정보를 제공하기를 원한다면 표준 예외를 확장해도 좋지만, 예외는 직렬화 할 수 있기 때문에 (Throwable은 Serializable을 구현하기 때문) 예외를 따로 커스텀하지 않는 것이 좋다.

## 애매한 예외 선택

* 주요 쓰임이 상호 배타적이지 않으므로, 재사용 가능 예외들 중 어떤 것을 선택해야 할 지 어려운 상황이 있다.
* 예를들어 인수 값이 무엇이었든 실패했을 거라면 IllegalStateException을, 아니라면 IllegalArgumentException을 던져야 한다.
