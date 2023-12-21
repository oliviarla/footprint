# Objects

{% hint style="info" %}
java.util 패키지에 존재하는 Objects 클래스는 객체에 대한 작업을 수행하거나 작업 수행 전 특정 상태를 확인하기 위한 정적 유틸리티 메서드로 구성된다.

해시코드 계산, 문자열 반환, 두 객체 비교, 범위 값 확인 등을 null-safe하게 수행하도록 메서드를 제공한다.
{% endhint %}

## reqiureNonNull

* 객체를 입력받아 null이라면 NullPointerException을 발생시키고, null이 아니면 반환해주는 메서드이다.
* 언뜻 보면 객체를 사용하는 시점에 자연스럽게 NullPointerException이 발생하도록 하는거랑 차이가 없지 않나 생각이 들 수 있다.
* 하지만 requireNonNull 메서드 사용 시, null이 들어왔을 경우 바로 예외가 던져지므로 콜스택이 간결해져 어느 부분에서 예외가 발생했는지 파악하기가 편하다.

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

## isNull / nonNull

* Java 8 lambda filtering을 위한 메서드이다.&#x20;
*   아래와 같이 filter 메서드의 인자로 `Predicate<? super T> predicate` 를 주어야하는 상황일 때 사용한다.

    > Predicate 함수형 인터페이스는 제네릭 타입의 객체를 입력받아 boolean을 반환하도록 한다.

```java
.stream().filter(Objects::isNull)
```

* `Objects::isNull` 대신 `x -> x == null` 형태로 작성할 수 있지만, 이펙티브 자바에 의하면 메서드 참조 방식이 일반적으로 더 간결하므로 권장한다고 한다.

{% content-ref url="../effective-java/7/item-43.md" %}
[item-43.md](../effective-java/7/item-43.md)
{% endcontent-ref %}

