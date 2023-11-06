# item 39) 명명 패턴보다 어노테이션을 사용하라

## 명명 패턴

* 전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소를 이름으로 구분하는 패턴
* 예를 들면, JUnit에서 버전 3까지 테스트 메서드의 이름을 반드시 test로 시작하도록 지어야 한다는 규칙이 있었다.

### 단점

* 오타가 나면 제대로 작동되지 않지만 오류를 발생시키지 않는다.
  * 비정상적인 상황을 정상적으로 간주하는 문제가 발생할 수 있다.
* 올바른 프로그램 요소에서만 사용되리라 보증할 수 없다.
  * 개발자의 의도에 따라 움직이지 않는 경우가 발생한다.
  * TestSafety... 라는 클래스를 만들고 이 클래스 내부의 모든 메서드가 JUnit에서 실행되길 기대해도, 실제로는 실행되지 않는다.
* 프로그램 요소를 매개변수로 전달할 수 없다.
* 원하는 예외가 발생해야 성공하도록 테스트 코드를 작성하는 경우, 메서드 이름에 예외의 이름을 덧붙여야 하지만 컴파일러가 해당 클래스가 존재하는지조차 확인하기 어렵다.

## 어노테이션 타입

* 원하는 조건을 만족하지 못할 경우 컴파일에러를 발생시키므로, 명명 패턴의 문제점을 해결해준다.
* 메서드나 클래스 위에 붙여 자동으로 로직이 수행되도록 해준다.

### **마커 어노테이션**

* 마커 어노테이션: 아무 매개변수 없이 단순히 대상에 마킹하는 용도로 사용하는 애너테이션
* 메타 어노테이션: 어노테이션 선언에 다는 어노테이션, 어노테이션의 조건을 정의할 수 있다.
* 아래는 마커 어노테이션으로 `@Test` 를 선언하는 예제이다.
* 메타 어노테이션을 통해 `@Test` 어노테이션이 1) Runtime에도 유지되어야 하며, 2) 반드시 메서드 선언에만 사용되어야 함을 의미하게 된다.

```java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적 메서드 전용이다.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{}
```

### **마커 어노테이션 처리기**

* 위 마커 어노테이션은 어떠한 로직을 처리하지 않는 단순한 어노테이션이다.
* 따라서 아래와 같이 **마커 어노테이션 처리기 클래스**를 두어 로직이 수행되도록 한다.
* reflection을 사용해 어노테이션이 붙은 메서드를 찾아 실행하고, 발생한 예외를 처리한다.

```java
public class RunTests {
  public static void main(String[] args) throws Exception {
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethods()) {
      if (m.isAnnotationPresent(Test.class)) {
        try {
          m.invoke(null);// @Test 어노테이션이 붙은 메서드 호출
        } catch (InvocationTargetException wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + " 실패: " + exc);
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
  }
}
```

### 매개변수가 있는 어노테이션

* 아래와 같이 어노테이션의 매개변수를 입력받을 수 있다.
* n개의 예외 클래스를 입력받아, 테스트 메서드에서 해당 예외가 발생했는지 확인하도록 한다.

> 모든 예외는 Throwable을 확장하므로 한정적 타입 토큰을 활용할 수 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  // 1) 하나의 매개변수를 입력받는 경우
  Class<? extends Throwable> value();
  // 2) 여러개의 매개변수를 배열으로 입력받는 경우
  Class<? extends Throwable>[] value();
}
```

* 마커 어노테이션 처리기 클래스에서 아래와 같이 매개변수를 가져와 원하는 예외가 발생했는지 확인할 수 있다.

```java
Class<? extends Throwable>[] excTypes =
        m.getAnnotation(ExceptionTest.class).value();
for (Class<? extends Throwable> excType : excTypes) {
    if (excType.isInstance(exc)) {
        break;
    }
}
```

### @Repeatable 메타 어노테이션

* 어노테이션에 `@Repeatable`을 달아 여러개의 값을 받는 어노테이션을 만들 수 있다.
* 아래와 같이 같은 **어노테이션에 매개변수를 다르게 입력해 여러번** 달 수 있다.

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { }
```

* 코드 가독성을 개선할 수 있다.
* 어노테이션 선언부와 처리하는 부분의 코드 양이 늘어난다.
* 처리 코드가 복잡해지므로 오류 발생할 가능성이 커진다.

#### **주의할 점**

1. @Repeatable을 명시한 어노테이션을 반환하는 컨테이너 어노테이션을 하나 더 정의하고, @Repeatable에 해당 컨테이너 어노테이션의 class 객체를 매개변수로 전달해야 한다.
2. 컨테이너 어노테이션은 내부 어노테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
3. 적절한 @Retention과 @Target을 명시해야 컴파일 오류가 발생하지 않는다.

#### @Repeatable 구현 예제

* 아래 예제는 ExceptionTest에 @Repeatable을 달고, 컨테이너 어노테이션을 매개변수로 전달하는 형태이다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```

* 마커 어노테이션 처리기 클래스에서 아래와 같이 매개변수를 가져와 예외가 발생했는지 확인할 수 있다.
* 어노테이션을 여러개 달면 해당 컨테이너 어노테이션 타입이 적용되므로, isAnnotationPresent 메서드로 컨테이너 어노테이션을 검사해야 한다.

```java
for (Method m : testClass.getDeclaredMethods()) {
    if (m.isAnnotationPresent(ExceptionTest.class)
            || m.isAnnotationPresent(ExceptionTestContainer.class)) {
        try {
            m.invoke(null);
            System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (Throwable wrappedExc) {
            Throwable exc = wrappedExc.getCause();
            int oldPassed = passed;
            ExceptionTest[] excTests =
                    m.getAnnotationsByType(ExceptionTest.class);
            for (ExceptionTest excTest : excTests) {
                if (excTest.value().isInstance(exc)) {
                    passed++;
                    break;
                }
            }
            if (passed == oldPassed)
                System.out.printf("테스트 %s 실패: %s %n", m, exc);
        }
    }
}
```
