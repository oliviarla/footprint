# 중재자 패턴

## 접근

* 프로그램이 발전함에 따라 다양한 객체 사이의 복잡한 통신과 제어를 한 곳으로 몰아 관리하고 싶을 때 중재자라는 클래스를 만들면 된다.
* 각각의 객체가 할 일을 정리하고, 상태가 바뀔 때 중재자에게 이를 알린다. 그리고 중재자가 요청을 보내면 응답을 해주어 전체적인 동작 규칙은 중재자가 담당하고, 각 클래스는 각각이 맡은 행위만 처리하면 된다.
* 가장 적절한 비유는 조종사들과 관제탑이다. 조종사는 다양한 객체에 해당되고, 관제탑은 중재자 객체에 해당된다.

## 개념

* 독립적으로 작동해야 하는 컴포넌트 간의 모든 직접 통신을 중단한 후, 중재자 객체를 호출하여 간접적으로 다른 컴포넌트로 리다이렉트되도록 한다.
* 이를 통해 컴포넌트들은 직접 수십 개의 동료 컴포넌트들과 결합되는 대신 단일 중재자 클래스에만 의존한다.
* **컴포넌트**들은 비즈니스 로직을 포함하는 다양한 클래스들이며 중재자 인터페이스에 대한 참조를 갖는다.
* **중재자 인터페이스**는 컴포넌트와의 통신 메서드를 선언한다.
* **중재자 구현체**는 다양한 컴포넌트 간의 관계를 캡슐화하기 위해 모든 컴포넌트에 대한 참조를 유지하고 특정 요청이 들어오면 이를 다른 컴포넌트들로 리다이렉트한다.

<figure><img src="../../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

* 옵저버 패턴과 유사하지만 중재자 패턴의 주목적은 컴포넌트 집합 간의 상호 의존성을 제거하는 것이고, 옵저버 패턴의 목적은 객체들 사이에 단방향 연결을 설정하는 것이 목적이다.

## 장단점

* 장점
  * 시스템과 객체를 분리해 재사용성을 향상시킨다.
  * 제어 로직을 한 곳에 모아 관리가 수월하고 단일 책임 원칙을 따른다.
  * 시스템에 속한 객체 간에 오가는 메시지를 줄이고 단순화할 수 있다.
* 단점
  * 중재자 객체에 너무 많은 로직이 몰려 복잡해질 수 있다.

## 예시

* 버튼, 선풍기, 전원공급기라는 클래스가 있고 이들 간에 서로 직접 통신하는 대신 중재자 클래스를 두어 전체적인 비즈니스 로직을 한 군데에 모아둘 수 있다.

```java
public class Mediator {
    private Button button;
    private Fan fan;
    private PowerSupplier powerSupplier;

    // constructor, getters and setters

    public void press() {
        if (fan.isOn()) {
            fan.turnOff();
        } else {
            fan.turnOn();
        }
    }

    public void start() {
        powerSupplier.turnOn();
    }

    public void stop() {
        powerSupplier.turnOff();
    }
}
```

```java
public class Button {
    private Mediator mediator;

    // constructor, getters and setters

    public void press() {
        mediator.press();
    }
}

public class Fan {
    private Mediator mediator;
    private boolean isOn = false;

    // constructor, getters and setters

    public void turnOn() {
        mediator.start();
        isOn = true;
    }

    public void turnOff() {
        isOn = false;
        mediator.stop();
    }
}
```

**출처**

* [https://refactoring.guru/ko/design-patterns/mediator](https://refactoring.guru/ko/design-patterns/mediator)
* [https://www.baeldung.com/java-mediator-pattern](https://www.baeldung.com/java-mediator-pattern)
