# item 30) 이왕이면 제네릭 메서드로 만들라

## **제네릭 메서드**

<figure><img src="../../../.gitbook/assets/image (41) (1).png" alt=""><figcaption></figcaption></figure>

```tsx
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);

    return result;
}
```

* 로타입을 사용하는 대신 제네릭 메서드를 사용하면, 직접 형변환하지 않아도 오류나 경고 없이 컴파일 된다.

## **제네릭 싱글턴 팩토리**

* 불변 객체를 여러 타입으로 활용할 수 있도록 할 경우
* 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 가능
* 요청된 여러가지 타입 매개변수에 맞게 매번 객체의 타입을 바꿔주는 정적 팩토리를 사용하는 방식이다.
* Collections.reverseOrder같은 함수 객체(item 42)나 emptySet같은 컬렉션용으로 사용한다.

## **항등 함수**

```swift
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

* 입력값을 수정없이 그대로 반환하는 함수이므로 T가 어떤 타입이든 형변환해도 타입 안전하다.

## **재귀적 타입 한정 명시**

* 제네릭 타입에 자기 자신이 들어간 표현식을 사용해 타입 매개변수의 허용 범위를 한정할 수 있다.
* `<E extends Comparable<E>>` 는 모든 타입 E는 자기 자신과 비교할 수 있다(상호 비교가 가능하다)는 의미이다. 예를들면 E에 String이 들어왔다면, String끼리 비교할 수 있다는 의미이다.

```tsx
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result Object.requireNonNull(e);

    return result;
}
```
