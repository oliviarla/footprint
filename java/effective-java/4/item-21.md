# item 21) 인터페이스는 구현하는 쪽을 생각해 설계하라

## **디폴트 메서드**

* 자바 8 이전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 수 없었다.
* 자바 8 이후부터 기존 인터페이스에 메서드를 추가할 수 있게 되었다.
* 디폴트 메서드 선언 후, 인터페이스를 구현한 클래스 중 해당 디폴트 메서드를 재정의하지 않은 모든 클래스가 디폴트 메서드를 사용
* 람다를 활용하기 위해 핵심 Collection 인터페이스들에 다수의 디폴트 메서드가 추가되었다.
* 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하는 것은 어려움

## **Collection 인터페이스의 removeIf 디폴트 메소드**

* 모든 원소를 순회하면서 주어진 predicate 함수가 true를 반환하면 반복자의 remove 메서드를 호출해 원소를 제거하는 디폴트 구현이 있다.

```php
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
	boolean result = false;
	for (Iterator<E> it = iterator(); it.hasNext(); ) {
		if (filter.test(it.next())) {
			it.remove();
			result = true;
		}
	}
	return result;
}
```

* 모든 메서드에서 클라이언트가 제공한 락 객체로 동기화한 후 내부 Collection 객체에 기능을 위임하는 SynchronizedCollection 클래스의 경우 removeIf 메서드를 재정의하지 않고 있다. 따라서 이 메서드를 사용하게 되면 락 객체를 사용할 수 없어 오류 발생할 수 있다.

## **인터페이스 설계법**

* 컴파일 에러가 발생하지 않아도 기존 구현체에 런타임 오류를 일으킬 수 있다.
* 기존 인터페이스에 디폴트 메서드를 새로 추가하는 일은 가급적 피하는 것이 좋다.
* 새로운 인터페이스를 만드는 경우 표준적인 메서드 구현 제공에 유용하고, 인터페이스를 더 쉽게 구현해 활용할 수 있다.
* 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도로 사용할 경우 기존 클라이언트를 망가뜨릴 수 있다.
* 인터페이스를 새로 릴리즈하기 전 최소 세 가지 이상의 구현을 해보고, 해당 인스턴스를 다양한 작업에 활용하는 클라이언트도 여러 개 만들어 봐야 한다.
