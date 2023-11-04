# item 32) 제네릭과 가변 인수를 함께 사용

## 가변 인수의 실수

* 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어지는데, 내부로 감추지 못하고 클라이언트에 노출되고 있다.
* 때문에 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면, 컴파일 경고가 발생한다.

## 제네릭과 가변 인수

* 거의 모든 제네릭과 매개변수화 타입은 실체화 불가 타입으로, 런타임에는 타입 정보가 소거된다.
* 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.

> 힙오염: 객체가 특정 제네릭 타입 인스턴스에 속해야하지만 실제로는 다른 인스턴스에 속하는 현상

```java
static void dangerous(List<String>... stringLists) {
        List<Integer> intList = List.of(3);
        Object[] objects = stringLists;//제네릭 배열으로 가변 매개변수를 받을 수 있다.
        objects[0] = intList;// 매개변수 배열에 무언가를 저장하여 힙 오염이 발생한다.
        String s = stringLists[0].get(0);// ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String
    }
```

* 제네릭과 varargs를 혼용하면 **타입 안정성**이 깨지므로 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.
* 제네릭 배열을 프로그래머가 직접 생성하는 건 허용하지 않고 제네릭 varargs 매개변수를 받는 메서드를 선언할 수 있게 한 이유는 실무에서 매우 유용하기 때문이다.
* 자바 라이브러리에서도 이러한 메서드를 제공한다. ex) Arrays.asList(T... a), Collections.addAll(Collection\<? super T> c, T... elements), EnumSet.of(E first, E...rest)

## @SafeVarargs

* 자바7에서 추가된 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있는 어노테이션
* 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이다.
* 메서드가 안전함을 확신할 수 있을 때는 해당 메서드가 순수하게 인수들을 전달하는 일만 할 때 안전하다.
  * 가변 인수 메서드를 호출 시 varargs매개변수를 담는 제네릭 배열이 만들어진다.
  * 가변 인수 메서드를 호출 시 varargs 매개변수를 담는 배열에 아무것도 저장하지 않을 때
  * varargs 배열의 참조가 밖으로 노출되지 않을 때

## 타입 안전성을 깨는 경우

```java
static <T> T[] toArray(T... args) {
    return args;
}
```

* toArray 메서드와 같이 varargs 매개변수 배열을 그대로 반환하면, 힙 오염이 메서드 호출한 쪽으로까지 전이된다.

```java
static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return toArray(a, b);
    case 1: return toArray(b, c);
    case 2: return toArray(c, a);
    }
    throw new AssertionError();// 도달할 수 없다.
}
```

* pickTwo 메서드는 매개 변수 타입이 String이라면 String\[] 타입을 반환하지 않고, 항상 어떤 타입의 객체를 넘기더라도 담을 수 있는 Object\[] 배열을 반환한다. toArray 메서드에서 반환되는 배열의 타입이 Object\[]이기 때문이다.
* 따라서 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다.

## 제네릭 varargs 매개변수를 안전하게 사용하는 방법

* 임의 개수의 리스트를 인수로 받아, 받은 순서대로 그 안의 모든 원소를 하나의 리스트로 옮겨 담아 반환한다.

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
    result.addAll(list);
    return result;
}
```

* 혹은 제네릭이나 매개변수화 타입의 **varargs 매개변수를 받는 모든 메서드를 안전하게 만든 후 @SafeVarargs 를 달아 컴파일러 경고를 없애도록** 한다.
* 1\) varargs 매개변수 배열에 아무것도 저장하지 않는지, 2) varargs 배열을 신뢰할 수 없는 코드에 노출되었는지 확인해 안전한 메서드로 만들자.

## **List매개변수**

* 혹은 varargs 매개변수를 List 매개변수로 바꾸어 타입 안전성을 검증할 수 있다.
* 이 경우 SafeVarargs 어노테이션이 필요 없고 안전성 체크를 할 필요가 없다.

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
    result.addAll(list);
    return result;
}
```

> 사용할 때에는 flatten(List.of("a","b","c")) 와 같이 사용할 것
