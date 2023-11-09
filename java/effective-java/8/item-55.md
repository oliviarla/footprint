# item 55) 옵셔널 반환은 신중히 하라

## 값을 반환할 수 없는 경우

* 자바 8 이전에는 Optional이 없었기 때문에 아래와 같은 방식을 사용했다.
  * 예외던지기 : 에외 생성시 스택 추적 전체를 캡쳐해야하므로 비용 문제
  * null 반환 : NullPointException을 막기 위한 null 처리 코드가 클라이언트에서 구현되어야 한다.
* 자바 8 이후에는 Optional 기능이 추가되었다.
  * 아무것도 반환하지 않아야 할 때, Optional\<T> 반환하도록 한다.
  * **예외를 던지는 것보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다**는 장점이 있다.

> Optional 객체: 원소를 최대 1개 가질 수 있는 ‘불변’ 컬렉션

* 아래 예제처럼 ofNullable 메서드를 사용해 null이 들어와도 안전히 반환하도록 처리할 수 있다.

```java
public static <E extends Comparable<E>>
	Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();// 빈 옵셔널 만들기

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);// 1) result가 null이면 NullPointerException 발생하는 메서드이다.
    return Optional.ofNullable(result);// 2) null값이 들어와도 안전하게 처리한다.
}
```

* Optional을 반환하는 메서드에서는 절대 null이 반환되지 않도록 해야 한다.
* 옵셔널은 검사 예외(item 71)의 취지와 비슷하다. 반환값이 없을 수 있음을 명확히 알려준다.

## 메서드가 옵셔널 반환할 때 클라이언트가 할 일

* 값을 못받을 경우의 기본값 설정하기

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

* 상황에 맞는 예외 던지기
  * 실제 예외가 아니라 예외생성 비용이 안든다.

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

* 항상 값이 채워져있다면 get 메서드로 값을 꺼내 사용한다. 값이 없다면 NoSuchElementException가 발생한다.

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

* 기본값 설정 비용이 너무 크다면 orElseGet() 사용하기
* orElseGet은 Optional 값이 존재하지않으면, 인수로 전달된 Supplier의 결과 값을 반환한다.

```java
public T orElseGet(Supplier<? extemds T> other){
	return value != null ? value : other.get();
}
```

* isPresent()를 통해 옵셔널 채워있다면 true, 비어있다면 false를 반환받아 사용할 수 있지만, 앞서 언급한 메서드들로 대체하는 것이 간결하고 용법에 맞는 코드가 된다.

## Optional과 Stream

* Optional을 Stream으로 변환해주는 어댑터

```java
// (자바8)
// 옵셔널에 값이 있으면 그 값을 꺼내서 (Optional::get)스트림에 매핑
streamOfOptionals.filter(Optional::isPresent).map(Optional::get)

// stream을 사용한다면 (자바9)
// 옵셔널에 값이 있으면 그 값을 담은 스트림 변환 / 없으면 빈 스트림 변환
streamOfOptionals.flatMap(Optional::stream)

```

## Optional 주의점

* 컬렉션, 스트림, 배열, Optional 같은 **컨테이너 타입은** Optional로 감싸기보단 **빈 컬렉션을 반환**하는 것이 비용이 적게 든다.
* 박싱된 기본 타입을 Optional로 감싸면 값을 두번이나 감싸기 때문에 기본타입보다 무거워진다. Optional에서 제공하는 클래스를 사용하자. (OptionalInt, OptionalLong, OptionalDouble)
* 옵셔널을 컬렉션의 키, 값, 배열의 원소로 사용하면 안된다. 값 자체가 없는 경우, 속이 빈 옵셔널일 경우로 값이 없다는 것을 나타내는게 복잡해진다.
* 필수가 아닌 필드 값을 표현할 때 값이 없음을 나타내기 위해 Optional로 선언하거나 반환하면 좋다.
