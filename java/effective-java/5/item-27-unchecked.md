# item 27) unchecked 경고를 제거하라

* 가능한 한 unchecked 경고를 모두 제거해 타입 안전성을 보장하도록 하자.

> 다이아몬드 연산자(<>): 이 연산자를 사용하면 타입 매개변수를 따로 명시할 필요 없고, 컴파일러가 자동으로 올바른 실제 타입 매개변수를 추론해준다.

* 경고를 제거할 수 없지만 타입이 안전하다고 확신되면, `@SuppressWarnings("unchecked")` 어노테이션을 붙여 경고를 숨길 수 있다. 하지만 런타임에 ClassCastException이 발생할 수 있다.
* `@SuppressWarnings` 어노테이션은 가능한 한 좁은 범위에 적용해야 한다. 변수 선언하는 쪽에 적용하는 것이 가장 좋고, 짧은 메서드 혹은 생성자까지는 사용할 수 있다.
* 아래 예시는 그대로 반환하는 대신 변수로 선언하여 어노테이션을 적용하는 것이다.

```kotlin
public <T> T[] toArray(T[] a) {
    if (a.length < size){
        // 생성한 배열과 매개변수로 받은 배열이 모두 T[]로 같으므로
        // 올바른 형변환이다.
        @SuppressWarnings("unchecked")
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

* 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.
