# item 66) 네이티브 메서드는 신중히 사용하라

## JNI란?

* 자바 네이티브 인터페이스
* 자바 프로그램이 네이티브 메서드를 호출하는 기술이다.

> 네이티브 메서드: C, C++ 같은 네이티브 언어로 작성한 메서드

### 사용하는 이유

* 레지스트리 같은 플랫폼 특화 기능을 사용하기 위함
* 네이티브 코드로 작성된 기존 라이브러리를 사용하기 위함 ex) 레거시 데이터를 사용하는 레거시 라이브러리
* 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 따로 네이티브 언어로 작성하기 위함 (JVM의 발전으로 인해 대부분 작업에서 다른 플랫폼에 견줄만한 성능을 보이기 때문에 권장하지는 않는다.)
* 자바가 하부 플랫폼의 기능들을 흡수하여(ex. OS 프로세스 접근 등) 네이티브 메서드의 필요성이 점점 줄고 있다.

### 네이티브 메서드 단점

* 자바보다 플랫폼을 많이 타서 이식성이 낮고, 디버깅이 어렵다.
* GC가 네이티브 메모리를 추적도 못하고 회수도 못한다.
* 자바, 네이티브 코드의 경계를 넘나 들때마다 비용이 추가된다.
* 네이티브 메서드와 자바 코드 사이의 '접착 코드'를 작성해야 하는데, 이는 귀찮고 가독성도 떨어진다.