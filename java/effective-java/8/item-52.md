# item 52) 다중정의는 신중히 사용하라

## 다중 정의 시 주의점

> 다중 정의(overloading): 이름이 같은 메서드가 매개변수의 타입이나 개수만 다르게 갖는 형태

* API 사용자가 매개변수를 넘길 때, 어떤 다중정의 메서드가 호출될지 모른다면 프로그램은 오작동하기 쉽다.
* 아래 예제처럼 메서드를 다중정의(overloading)할 경우 **어느 메서드를 호출할 지 컴파일 타임에 정해진다**.
* **다중정의**한 메서드는 **정적으로 선택**되기 때문에, 런타임에 매번 타입이 달라져도 컴파일 타임에 정해진 메서드를 사용하여 호출할 메서드가 바뀌지 않는다. (이와 같이 헷갈리는 코드는 작성하지 않는게 좋다)

```java
public static String classify(Set<?> s) {
    return "집합";
}

public static String classify(List<?> lst) {
    return "리스트";
}

public static String classify(Collection<?> c) {
    return "그 외";
}

public static void main(String[] args) {
    Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
    };

    for (Collection<?> c : collections)
        System.out.println(classify(c));
}
```

* 메서드를 합친 후 instanceof로 명시적으로 검사를 수행해야 한다.

```java
public static String classify(Collection<?> c) {
    return c instanceof Set  ? "집합" :
            c instanceof List ? "리스트" : "그 외";
}
```

* 안전하고 보수적으로 가려면, 매개변수 수가 같은 다중정의는 만들지 말 것
* 가변 인수를 매개변수로 사용한다면, 다중정의는 사용하면 안 된다.
* 이왕이면, 이름에 의미를 넣어 다르게 지어주는 것이 좋다.

## 재정의 vs 다중정의

* 하위 클래스에서 **재정의**한 메서드는 **동적으로 선택**되기 때문에, 컴파일타임 타입에 무관하게 가장 하위에서 정의한 재정의 메서드가 실행된다.
* 다중정의 메서드는 정적으로 선택되기 때문에 예외적인 동작처럼 보일 수 있다.

## 생성자의 다중정의

* 생성자의 이름을 다르게 지을 수 없으므로 두 번째 생성자부터는 무조건 다중정의가 된다.
* 생성자는 재정의할 수 없으니 다중정의와 재정의가 혼용될 걱정도 없다.
* 여러 생성자가 같은 수의 매개변수1를 받아야 하는 경우 등 혼돈이 생길 수 있다면, **정적 팩터리 메서드**를 사용하면 좀 더 직관적일 수 있다.

## 다양한 다중정의 케이스

* 메서드 다중정의 시 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받으면 안된다.
* 참조된 메서드와 호출한 메서드 둘 다 다중정의되었을 경우 다중정의 해소 알고리즘이 동작하지 않는 상황이 발생할 수 있다.
* 어떤 다중정의 메서드가 불리는지 몰라도 기능이 같다면 신경쓰지 않아도 된다.
* 아래와 같이 더 특수한(혹은 여러 개의 인자를 갖는) 다중정의 메서드에서 덜 특수한 메서드로 포워드하면 동일한 일을 하도록 구현할 수 있다.

```java
public boolean contentEquals(StringBuffer sb) {
	return contentEquals((CharSequence) sb);
}
```
