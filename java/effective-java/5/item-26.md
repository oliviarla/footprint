# item 26) 로 타입은 사용하지 말 것

## **제네릭**

* 자바 5부터 사용할 수 있다.
* 컬렉션에서 객체를 꺼낼 때마다 형변환이 필요하다.
* 제네릭을 사용하면 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주고, 컴파일러는 알아서 형변환 코드를 추가하고 담을 수 있는 타입만 사용하도록 하기 때문에 안전하고 명확한 프로그램을 만들어준다.
* 제네릭 클래스/ 제네릭 인터페이스: 클래스와 인터페이스 선언에 타입 매개변수가 쓰인 경우 ex) List 인터페이스는 List\<E>와 같이 타입 매개변수를 받는다.
* 제네릭(Generic) 타입: 제네릭 클래스와 제네릭 인터페이스를 통틀어 말한다. 각 제네릭 타입은 매개변수화 타입을 정의한다. 매개변수화 타입인 List\<String>의 String 부분이 실제 타입 매개변수이다.

## **로(raw) 타입**

* 제네릭타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.
* 제네릭이 사용되기 전의 하위호환성 지원을 위해 남겨둔 것
* 잘못된 타입을 입력해도 컴파일 오류는 나지 않고 런타임에서 오류가 나게 된다.
* 로 타입 대신 매개변수화된 타입을 사용하면, 다른 타입의 인스턴스를 넣으려 할 때 컴파일 오류가 나므로 디버깅이 훨씬 쉽다.

```cpp
private final Collection<Stamp> stamps = ...;
stamps.add(new Coin()); // 컴파일 오류 발생
```

## **로 타입과 Collection\<Object>의 차이**

* 매개변수로 List를 받는 메서드에 List\<String>을 넘길 수 있지만, List\<Object>를 받는 메서드에는 제네릭의 하위 규칙 때문에 List\<String>을 매개변수로 넘길 수 없다.
* raw 타입 List를 받는 메서드는 컴파일 에러는 발생하지 않지만 실행 시 ClassCastException이 발생한다.
* generic 타입 List를 받는 메서드는 컴파일 시 타입 에러가 발생하여 실행이 불가능하다.

```tsx
public class Test {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();

        // raw 타입 -> 실행 시 ClassCastException 발생
        unsafeAddRaw(strings, Integer.valueOf(5));
        String s = strings.get(0);

        //generic 타입 -> 컴파일 시 타입 에러 발생
        unsafeAddGeneric(strings, Integer.valueOf(5));
        String s = strings.get(0);
    }

    private static void unsafeAddRaw(List list, Object o) {
        list.add(o);
    }

    private static void unsafeAddGeneric(List<Object> list, Object o) {
        list.add(o);
    }
}
```

## **비한정적 와일드카드 타입**

* 로 타입 컬렉션에는 아무 원소나 넣을 수 있어 타입 불변식을 훼손하기 쉽다.
* 실제 타입 매개변수를 몰라도 제네릭을 사용하고 싶을 때 `<?>` 을 사용한다.
* 예를 들어 `Set<?>`은 어떤 타입이라도 담을 수 있는 범용적인 매개변수화 Set 타입이다.
* 로 타입 컬렉션과 달리 `Collection<?>`에는 null을 제외한 어떤 원소도 넣을 수 없다. 즉 불변식을 지킬 수 있다.
* 예를 들어 어떤 실제 타입 매개변수를 가지는 Set들을 사용하고 있는데, 특정 메서드의 매개변수로 Set\<?>을 입력받도록 하여 **어떤 타입의 Set이라도 컬렉션 조회/순회**가 가능하다. 그리고 메서드 내부에서 컬렉션에 원소를 추가를 할 수 없으니 **불변식을 지킬 수 있는 효과**도 있다.

## **로 타입을 사용하는 경우**

* 클래스 리터럴은 로 타입을 사용해야 한다. 예를 들어 List.class, String\[].class, int.class는 허용되지만, List\<String>.class, List\<?>.class는 사용 불가
* instanceof 연산자는 비한정적 와일드카드 타입 외에 매개변수화 타입에는 적용할 수 없다. **런타임(Runtime)**&#xC5D0; 제네릭 타입 정보가 지워지기 때문이다. 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 똑같이 동작하므로, 코드가 간결하도록 로 타입을 사용하도록 하자.

```jsx
if( o instanceof Set) {
    Set<?> s = (Set<?>) o;//와일드카드 타입으로 형변환
}
```
