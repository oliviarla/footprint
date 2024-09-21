# 어댑터 패턴

## 접근

* 어댑터(adaptor)는 서로 다른 국가마다 채택하는 플러그와 전원 소켓이 다를 때 플러그를 다른 전원 소켓 타입에도 꽂을 수 있도록 인터페이스를 바꿔주고 필요 시 전압도 변환해주는 역할을 한다.
* 객체 지향에서 어댑터라는 개념을 도입하여 어떠한 인터페이스를 특정 클라이언트가 요구하는 형태로 적응시킬 수 있게 해줄 수 있다.

## 개념

* **특정 클래스 인터페이스를 클라이언트에서 요구하는 다른 인터페이스로 변환**한다.
* 인터페이스가 호환되지 않아 사용 불가능한 클래스를 사용할 수 있도록 해준다.
* **클라이언트**는 타겟 인터페이스에 맞게 구현되어 있다. **어댑터**는 타겟 인터페이스를 구현하며 어댑티 인스턴스를 필드로 가진다. **어댑티** 인터페이스는 타겟 인터페이스와 다른 형태이다.
  * 어댑터는 어댑티를 컴포지션 형태로 감싸는 형태이다.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

* 특정 구현이 아닌 인터페이스에 연결하는 형태이므로 서로 다른 클래스로 변환시키는여러 어댑터를 쓸 수 있다.&#x20;
* 클라이언트에서 타겟 인터페이스를 사용해 메서드를 호출하면 어댑터에 요청이 보내진다.
* 만약 어댑티 클래스에서 타깃 인터페이스의 기능을 제공할 수 없다면 `UnsupportedOperationException` 을 반환할 수 있다.
* 두 인터페이스를 모두 구현하는 다중 어댑터를 사용할 수도 있다. 혹은 하나의 어댑터 클래스에서 타깃 인터페이스를 구현하기 위해 2개 이상의 어댑티를 사용해야 하는 경우가 발생할 수 있다.
* 여태껏 다룬 내용은 **객체 어댑터**에 대한 것이며, **클래스 어댑터**는 다중 상속이 가능한 언어일 때 사용 가능하며, 어댑터 클래스는 타깃과 어댑티를 모두 상속받아 구현된다.
* 데코레이터 패턴은 책임과 행동을 추가하는 것에 의의를 둔다. 어댑터 패턴은 인터페이스를 변환하는 것에 의의를 둔다. 두 패턴 모두 **클라이언트 코드를 고치지 않고** 새로운 행동을 추가하거나 새로운 구현체를 사용할 수 있다는 공통점이 있다.

## 장단점

* 장점
  * 인터페이스 또는 데이터 변환 코드를 담당하므로 단일 책임 원칙을 지킨다.
  * 클라이언트 코드가 클라이언트 인터페이스를 통해 어댑터 클래스를 사용하면 기존의 클라이언트 코드를 손상시키지 않고 새로운 유형의 어댑터들을 프로그램에 도입할 수 있으므로 개방 폐쇄 원칙을 지킨다.
* 단점
  * 새로운 인터페이스와 클래스들을 도입해야 하므로 코드의 복잡성이 증가한다. 코드의 나머지 부분과 호환되도록 서비스 클래스를 변경하는 것이 더 간단할 수 있다.

## 사용 방법

* 어댑터 클래스를 만들어 어댑티 객체를 필드로 두고, 타겟 인터페이스를 구현한다.
* 클라이언트 코드에서는 어댑터 객체를 인터페이스 필드에 대입해 사용하면 된다.

## 예시

* Duck 인터페이스를 사용하는 클라이언트에게 Duck 대신 Turkey 객체를 넘겨주고자 한다면 아래와 같이 어댑터를 사용해야만 인터페이스가 동일해져 사용 가능해진다.

```java
public interface Duck {
    public void quack();
    public void fly();
}

public interface Turkey {
    public void gobble();
    public void fly();
}
```

```java
public class TurkeyAdapter implements Duck {
    Turkey turkey;
    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }
    
    public void quack() {
        turkey.gobble();
    }
    
    public void fly() {
        for(int i=0; i<5; i++) {
            turkey.fly();
        }
    }
}
```

* Spring의 HandlerAdapter도 대표적인 어댑터 패턴의 예시이다.

```java
public interface HandlerAdapter {
    boolean supports(Object handler);
    
    @Nullable
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
}
```

```java
public class SimpleControllerHandlerAdapter implements HandlerAdapter {

    @Override
    public boolean supports(Object handler) {
      return (handler instanceof Controller);
    }

    @Override
    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {

      return ((Controller) handler).handleRequest(request, response);
    }
  }
```

