# 객체 지향 프로그래밍

{% hint style="info" %}
💡 객체지향의 키워드

* 상속과 조합
* 객체의 상태, 역할
* 메세지
* 다형성
* 의존 관계
* 설계의 유연성&#x20;
{% endhint %}

### 객체지향 프로그래밍

* 객체 지향 프로그래밍이란, 컴퓨터 프로그램을 명령어의 목록으로 보는 시각에서 벗어나 여러 개의 독립된 단위, 즉 "객체"들의 **모임**으로 파악하고자 하는 것
* 각각의 **객체**는 **메시지**를 주고받고, 데이터를 처리할 수 있다. (**협력 관계**)
*   객체 지향 프로그래밍은 프로그램을 **유연**하고 **변경**이 용이하게 만들기 때문에 대규모 소프트웨어 개발에 많이 사용된다.

    → 컴포넌트를 쉽고 유연하게 변경하면서 개발

### 다형성

* **역할**과 역할을 실행하는 **구현**으로 분리한다
* **클라이언트**는 대상의 역할(인터페이스)만 알면 된다.
* **클라이언트**는 구현 대상의 **내부 구조를 몰라도** 된다.
* **클라이언트**는 구현 대상의 **내부 구조가 변경**되어도 영향을 받지 않는다.
* **클라이언트**는 구현 **대상 자체를 변경**해도 영향을 받지 않는다. (새로운 구현으로 기존에 역할을 구현했던 것을 대체할 수 있음)
* 자바에서 역할은 인터페이스, 구현은 인터페이스의 구현 클래스/객체
* 인터페이스가 변경되면 이를 사용하는 부분을 크게 변경해야 하므로 인터페이스를 안정적으로 잘 설계하는 것이 중요

#### SOLID 원칙

* SRP: 단일 책임 원칙(single responsibility principle)
  * 한 클래스는 하나의 책임만 가져야 한다.
  * 하나의 책임이라는 것은 모호함, 범위와 문맥과 상황에 따라 다르다.
  * **중요한 기준은 변경**이다. 변경이 있을 때 파급 효과가 적으면 단일 책임 원칙을 잘 따른 것 예) UI 변경, 객체의 생성과 사용을 분리
* OCP: 개방-폐쇄 원칙 (Open/closed principle)
  * 가장 중요한 원칙
  * 소프트웨어 요소는 **확장에는 열려** 있으나 **변경에는 닫혀** 있어야 한다
  * 기존 코드를 변경하지 않아야 함
  * **다형성**을 활용해 인터페이스를 구현한 클래스를 만들어서 새로운 기능을 구현
  * 구현 객체를 변경하기 위해 클라이언트 코드의 변경이 필요한 경우 OCP 원칙을 위반하게 된다.
  * 따라서 **객체를 생성하고, 연관관계를 맺어주는 별도의 조립, 설정자가 필요**하다. (DI)
* LSP: 리스코프 치환 원칙 (Liskov substitution principle)
  * 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀수 있어야 한다
  * 하위 클래스는 인터페이스의 규약을 기능적으로 지켜서 구현해야 함
* ISP: 인터페이스 분리 원칙 (Interface segregation principle)
  * 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다
  * 기능별로 인터페이스를 분리해 설계할 것
* DIP: 의존관계 역전 원칙 (Dependency inversion principle)
  * **추상화에 의존해야지, 구체화에 의존하면 안된다.**
  * 대본과 상대 배역에 의존해야지, 특정 배우와 호흡만 의존해 다른 배우와는 연기를 못할 경우 안된다는 의미
  * `MemberRepository m = new MemoryMemberRepository();` 와 같이 사용자가 구체화된 Repository를 모두 알고 있어야 변경이 가능한 상태임

#### 스프링과 객체지향

* 스프링은 DI컨테이너를 제공해 다형성의 OCP, DIP를 가능하게 하도록 지원함
* **클라이언트 코드의 변경 없이 기능 확장**
* **쉽게 부품을 교체하듯이 개발**
* 인터페이스 도입 시 추상화라는 비용 발생
* 자세하게 파악하려면 구현클래스를 찾아 들어가야 함
* 기능을 확장할 가능성이 없다면, 구체 클래스를 직접 사용하고, 향후 꼭 필요할 때 리팩토링해서 인터페이스를 도입하는 것도 방법이다.
