# item 47) 반환 타입으로는 스트림보다 컬렉션이 낫다

## 스트림보다는 컬렉션을 반환

* 스트림은 반복을 지원하지 않으므로, 스트림과 반복 구문을 잘 조합해야 좋은 코드가 나온다.
* Stream 인터페이스는 Iterable 인터페이스의 추상 메서드를 모두 정의해 놓았지만 Iterable을 확장하지 않아 for-each문으로 반복할 수 없다.
* Stream를 Iterable로 중개해주는 어댑터 메서드를 사용하면 스트림을 for-each문으로 반복할 수 있다. 하지만 클라이언트 코드를 복잡하게 만들고, 성능이 느리다.
* Collection 인터페이스는 Iterable 인터페이스의 하위 타입이고, stream 메서드도 제공하니 일반적으로는 Collection을 반환하도록 한다.

## 스트림으로 반환해야 하는 경우

* Collection의 각 원소는 메모리에 올라가므로, 시퀀스의 크기가 크다면 전용 컬렉션을 구현하는 것이 성능 측면에서 좋다.
* Stream을 반환하면, 컬렉션을 반환하는 것보다 메모리 사용을 줄이는 효과가 있다.
* 반복 시작 전 시퀀스의 contains와 size 로직을 확정할 수 없는 경우에는 Stream을 반환하는 것이 좋다.
* 다음은 입력 리스트의 모든 부분리스트 반환하는 예제이다. prefixes와 suffixes 메서드가 있는데, (a, b, c)의 prefixes 결과는 (a), (a, b), (a, b, c) 이고, (a, b, c)의 suffixes 결과는 (c), (b, c), (a, b, c) 이다.
* 조금 복잡하지만 가독성이 좋은 버전

```java
public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(Item47::suffixes));
    }

    public static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }

    public static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.rangeClosed(0, list.size()-1)
                .mapToObj(start -> list.subList(start, list.size()));
    }

    public static void main(String[] args) {
        SubLists.of(List.of("a", "b", "c")).forEach(System.out::println);
    }
}
```

* 간결하지만 가독성이 안좋은 버전 (이중 for문을 그대로 stream화)

```java
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
        .mapToObj(start ->
                  IntStream.rangeClosed(start + 1, list.size())
                           .mapToObj(end -> list.subList(start, end)))
        .flatMap(x -> x);
}
```
