# item 29) 이왕이면 제네릭 타입으로 만들라

## **제네릭 타입 만들기**

* 클라이언트에서 직접 형변환해야 하는 Object 타입보다는 제네릭 타입이 더 안전하고 편리하다.
* 클래스 선언에 타입 매개변수를 추가한다 (보통 타입 이름으로 E를 사용한다, item68)
* Object 기반의 코드가 있다면, 적절한 타입 매개변수로 바꾼다.

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;// 다 쓴 참조 해제
        return result;
    }
    ... // isEmpty와 ensureCapacity 메서드는 그대로다.
}
```

* E와 같은 실체화 불가 타입으론 배열을 만들 수 없어 `new E[DEFAULT_INITIAL_CAPACITY];` 가 실패한다.

#### **해결방법**

*   방법 1) 제네릭 배열 생성 금지 제약을 우회하기

    * Object 배열 생성한 후 형변환을 한다. 대신 안전성이 보장되는지 직접 확인해야 한다.
    * 비검사 형변환의 안전함이 증명되었다면 `@SuppressWarnings` 어노테이션으로 경고를 숨긴다.

    ```java
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
    ```
*   방법 2) elements 필드의 타입을 Object 배열으로 변경

    * 배열에 입력하는 부분에서 E 타입만 허용한다면, 형변환이 안전하다.

    ```java
    public Stack() {
        @SuppressWarnings("unchecked") E result = (E) elements[--size];
    }
    ```

#### **배열보다 리스트를 우선하라면서(item28) 왜 여기서는 배열을 쓰는가?**

* 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다.
* 자바가 리스트를 기본 타입으로 제공하지 않아 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다.
* HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.
