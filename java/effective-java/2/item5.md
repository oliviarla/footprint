# item5) 자원을 직접 명시하는 대신 의존 객체 주입 사용

* 하나 이상의 자원에 의존하고 사용하는 자원에 따라 클래스의 동작이 달라지는 경우, **자원을 직접 final으로 명시하는 정적 유틸리티 클래스, 싱글톤 방식**은 **적합하지 않다.**
* 필요한 자원들을 클래스가 직접 만들게 하는 것도 좋지 않다.

## **의존 객체 주입**

* 인스턴스 생성 시 필요한 자원을 넘겨주는 방식
* 클래스가 여러 자원 인스턴스를 지원해야하고 클라이언트가 원하는 자원을 사용할 수 있도록 한다.
* 불변을 보장하여 같은 자원을 사용하려는 클라이언트들이 의존 객체들을 안심하고 공유할 수 있다.
* 생성자, 정적 팩토리, 빌더에 응용 가능하다.
* 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

## **생성자에 자원 팩토리를 넘겨주는 방식**

> 팩토리: 호출할 때 마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체, 팩토리 메서드 패턴을 구현한 것

* 팩토리를 생성자에 넘겨주어 생성자 내부에서 원하는 인스턴스를 생성해 사용 가능

## **Supplier\<T> 인터페이스**

* 클라이언트가 명시한 타입의 하위 타입을 생성할 수 있는 인터페이스를 팩토리로 넘겨받는다.
* 일반적으로 한정적 와일드카드 타입을 사용해 팩토리의 타입 매개변수를 제한해야 한다.
* 생성자에 Supplier 인터페이스를 넘겨주어 자원 팩토리를 넘겨주는 방식을 구현할 수 있다.
* 아래 코드는 tileFactory를 통해 Tile 클래스의 하위 타입 클래스를 얻어 Mosaic 클래스를 생성할 수 있도록 한다.

```jsx
Mosaic create(Supplier<? extends Tile> tileFactory) {...}
```