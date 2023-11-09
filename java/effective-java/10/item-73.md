# item 73) 추상화 수준에 맞는 예외를 던지라

## 예외 번역(exception translation)

* 상위계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.
* 만약 이 처리를 하지 않으면 수행하려는 일과 관련없어보이는 예외가 튀어나와 클라이언트 프로그램을 깨지게 할 수 있다.

```cpp
try {
    ...// 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```

## 예외 연쇄(exception chaining)

* 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식
* 별도의 접근자 메서드를 통해 필요할 때 저수준 예외를 꺼내 확인할 수 있다.

```scala
try {
    ...// 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
    throw new HighetLevelException(cause);
}

class HigherLevelException extends Exception {
//고수준 예외 연쇄용 생성자HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

* Throwable의 initCause 메서드를 이용해 원인을 직접 저장해둘 수 있다.
* 문제의 원인을 getCause 메서드로 프로그램에서 접근할 수 있다. 원인과 고수준 예외의 스택 추적 정보를 잘 통합해준다.

```coffeescript
try {
    ...
} catch (SpaceException e)	{
    InstallException ie = new InstallException("설치 중 예외발생");
    ie.initCause(e);
    throw ie;
} catch (MemoryException me) {
    InstallException ie = new InstallException("설치 중 예외발생");
    ie.initCause(me);
    throw ie;
}
```

## 정리

* 예외 번역을 남용하기보다는 아래 계층에서 예외가 발생하지 않도록 하는 것이 최선이다.
* 상위 계층에서 예외를 조용히 로깅 기능(java.util.logging) 등을 활용해 처리하여 문제를 API 호출자까지 전파하지 않도록 한다.
* 이를 통해 클라이언트 코드와 사용자에게 문제를 전파하지 않으면서도 로그를 분석해 추가 조치를 취할 수 있다.
