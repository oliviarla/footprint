# Assertions

## 사용 방법

* JDK 1.4 부터 특정 조건을 두고 만족하지 못할 경우 `AssertionError` 를 발생시킬 수 있는 `assert` 문을 사용할 수 있다.
* 아래와 같이 assert문 이후에 특정 조건만 명시하거나, 특정 조건과 함께 어떤 예외 메시지를 반환할 지 명시할 수 있다.

```java
assert condition;
assert condition : errormessage;
```

* 이전 버전과의 호환성을 위해 JVM은 기본적으로 Assertion 유효성 검사를 비활성화 한다. 따라서 애플리케이션 실행 시 `-ea` 옵션을 추가해주어야 동작한다.
* 아래와 같이 특정 클래스와 패키지에만 적용할 수 있도록 지정할 수 있다.

```
-ea Main : 시스템 클래스를 제외한 모든 클래스 활성화
-ea:<class name> Main : 지정한 클래스만 활성화
-ea:... Main : 기본 패키지를 활성화(모든 클래스)
-ea:<package name>... Main : 지정된 패키지를 활성화(모든 클래스와 하위 패키지 포함)
-ea -da:<class name> Main : 지정된 클래스를 제외한 모두 활성화
```

* `AssertionError` 는 Error를 상속받고, 결국 타고 올라가면 Throwable을 상속받는다. 따라서 Unchecked Exception에 해당하기 때문에 그대로 흘러가도록 두어야 하며 try-catch 하는 등 처리하는 것은 올바르지 못한 처리 방법이다.
* 이와 같은 특성으로 인해 AssertionError가 발생하면 프로그램이 종료될 수 있으므로 실제 배포 환경에서는 사용하기 어렵다.
* public 메서드의 입력값을 확인하는 경우에는 assertions 대신 IllegalArgumentException이나 NullPointerException과 같은 Unchecked Exception을 사용해야 한다.
* 조건문에서 다른 메서드를 직접 호출하지 말고, 미리 지역 변수에 결과를 할당한 후 해당 변수를 사용해야 한다.

**출처**

* https://baeldung.com/java-assert
* https://offbyone.tistory.com/294
