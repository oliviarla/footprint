# 전략 패턴

## 접근

* **변경되는 부분과 변경되지 않는 부분을 분리**하고, 세세한 **구현보다는 인터페이스(super type)에 맞추어 프로그래밍**하도록 해야 한다.
*   “A에는 B가 있다” 와 같은 관계의 경우 **컴포지션 형태**로 구현해야 한다.

    > 이 책에서 상속 대신 컴포지션을 활용하면 유연성을 크게 향상시킬 수 있으며 여러 디자인 패턴에서 쓰인다고 한다. 하지만 상속을 무조건 쓰지 않아야 하는 것은 아니며, 코드 재활용을 위해서 상속을 사용하는 것이 아니라 원하는 형태로 타입 계층을 만들기 위해서 사용한다면 문제 없다.

## 개념

* 알고리즘 패밀리를 정의하고 각 패밀리를 별도의 클래스에 넣어 캡슐화하여, 객체들을 사용자가 상호 교환할 수 있도록 한다.
* **전략**과 **컨텍스트**라는 용어를 사용한다.
* **전략**이란 공통된 목적을 갖는 알고리즘에 대한 인터페이스이며 다양한 구현체들이 존재할 수 있다.
* **컨텍스트**란 전략을 참조하며 원하는 기능을 제공하기 위해 전략에 알고리즘 수행을 위임한다.
* 클라이언트는 컨텍스트를 생성할 때 원하는 전략 구현체를 전달한다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 장단점

* 장점
  * 상속을 사용하지 않고도 코드의 중복을 줄일 수 있다.
  * 컨텍스트를 변경하지 않고도 다양한 전략들을 런타임에 유동적으로 사용할 수 있어 개방 폐쇄 원칙을 따른다.
* 단점
  * 알고리즘이 계속해서 늘어날 가능성이 없다면 구조가 복잡해지므로 사용할 필요가 없다.
  * 클라이언트가 전략을 선택하기 위해 각 전략에 대한 내용과 차이점을 알고 있어야 한다.
  * 많은 프로그래밍 언어에는 익명 함수(람다)를 사용해 알고리즘의 다양한 버전들을 구현할 수 있는 함수형 프로그래밍을 지원하므로, 인터페이스, 클래스가 따로 필요한 전략 패턴을 사용하지 않아도 된다.

## 사용 방법

### Strategy 필드 사용

* 내부에 Strategy 필드를 두어 Strategy 구현체를 주입하는 방식
* Strategy 의 구현체를 변경하거나 새로 만들어도 Context 코드에는 영향을 주지 않는다.
* 이 구현 방식은 Context와 Strategy를 실행 전에 원하는 모양으로 조립해두고 Context를 실행하는 **선 조립, 후 실행** 방식이다.
* 이로 인해 Context를 실행할 때 마다 Strategy 클래스를 변경하기 어렵다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Context 클래스

```java
@Slf4j
public class Context {
  private Strategy strategy;

  public Context(Strategy strategy) {
    this.strategy = strategy;
	}

	public void execute() {
		long startTime = System.currentTimeMillis();

		strategy.call();

		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime; 
		log.info("resultTime={}", resultTime);
	}
}
```

* 익명 내부 클래스로 Strategy 객체를 주입해 Context를 실행할 수 있다.

```java
Context context = new Context(new Strategy() {
  @Override
	public void call() {
		log.info("비즈니스 로직1 실행");
	}
});
context.execute();
```

* 람다를 사용해 Strategy 객체를 주입해 Context를 실행할 수 있다.

```java
Context context1 = new Context(() -> log.info("비즈니스 로직1 실행"));
context1.execute();

Context context2 = new Context(() -> log.info("비즈니스 로직2 실행"));
context2.execute();
```

### 파라미터로 Strategy 전달

* Context 를 실행할 때 마다 전략을 인수로 전달한다.
* 실행할 때 마다 전략을 유연하게 변경할 수 있다.
* 하나의 Context 객체만 만들어두고, 실행 시점에 여러 전략을 인수로 전달해 유연하게 실행할 수 있다.

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Context 클래스에 Strategy 필드를 두지 않는 대신, 파라미터로 전달된 Strategy를 사용한다.

```java
@Slf4j
public class Context {

		public void execute(Strategy strategy) {
				long startTime = System.currentTimeMillis();

				strategy.call();

				long endTime = System.currentTimeMillis();
				long resultTime = endTime - startTime;
				log.info("resultTime={}", resultTime);
		}

}
```

* 람다를 사용해 Strategy 객체를 주입해 Context를 실행할 수 있다.
* Context를 실행할 때 마다 Strategy를 넘기기 때문에 하나의 Context 객체를 재활용할 수 있다.

```java
ContextV2 context = new ContextV2();
context.execute(() -> log.info("비즈니스 로직1 실행"));
context.execute(() -> log.info("비즈니스 로직2 실행"));
```

## 예시

* 여러 게임 캐릭터가 존재하고, 해당 캐릭터가 사용할 수 있는 다양한 무기가 있는 경우를 설계할 때 전략 패턴을 사용할 수 있다.
* 일단은 게임 캐릭터에 대한 일반적인 특성을 Character 추상 클래스에 추가하고, 각 게임 캐릭터는 독립된 클래스(ex. King, Knight, ..)로 구성한다.
* 게임 캐릭터는 한 번에 하나의 무기를 사용해 싸울 수 있으므로 무기를 갖도록 Character 추상 클래스에 무기 관련 필드를 추가한다. 여기서 주목할 점은 **게임 캐릭터에는 무기가 있다**라는 점에서 컴포지션을 이용한 것이다.
* 무기의 경우 어떻게 상대에 데미지를 입히는지 구현이 각각 다르다. 따라서 Weapon 인터페이스를 추가하고, 이를 구현하는 다양한 무기 클래스(ex. Axe, Sword, …)를 구성한다.

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

**출처**

* 김영한 스프링 핵심 원리 강의
* 헤드퍼스트 디자인 패턴
* refactoring.guru
