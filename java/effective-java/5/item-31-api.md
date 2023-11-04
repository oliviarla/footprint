# item 31) 한정적 와일드카드를 사용해 API 유연성을 높이라

## 한정적 와일드카드 타입

* \<? extends E> 또는 \<? super E> 로 나타낸다.
* 매개변수화 타입은 불공변이다. 예를들면 List\<String>은 List\<Object>의 하위 타입이 아니다.
* 기존 제네릭 방식으로는 Stack\<Number> 에 하위 타입인 Integer를 넣는 것은 불가능하다.
* 아래와 같이 와일드카드 타입을 사용하면 Number에 하위 타입을 입력할 수 있다.
* Iterable\<? extends E>: E의 하위 타입의 Iterable이라는 의미

```php
public void pushAll(Iterable<? extends E> src){
    for(E e : src)
        push(e);
}
```

* E의 상위 타입의 Collection이어야 한다면 Collection\<? super E> 를 사용

```tsx
public void popAll(Collection<? super E> dst) {
    while(!isEmpty())
        dst.add(pop());
}
```

## PECS 공식

* 유연성을 극대화 하려면 원소의 producer나 consumer용 입력 매개변수에 와일드카드 타입을 사용할 것
* pushAll의 경우 Stack에 넣을 E 인스턴스를 생산하므로, producer이다.
* popAll의 경우 Stack으로부터 E 인스턴스를 꺼내 소비하는 입장이므로 consumer이다.
* **producer는 extends를, consumer는 super를 사용**해야 한다.
* 다음 두 타입은 항상 소비자이므로, Comparable\<? super E>, Comparator\<? super E> 를 사용하도록 한다.
* 이를 통해 Comparable이나 Comparator를 직접 구현하지 않았지만, 구현된 타입을 확장한 클래스/인터페이스를 지원할 수 있다.

## 메서드 정의 시 타입 매개변수 vs 와일드카드

* 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하도록 한다.
* 와일드 카드 사용 시 null 이외의 값을 추가할 수 없는데, 추가하고 싶을 때에는 private 도우미 메서드를 제네릭 메서드로 작성해 와일드카드 타입의 실제 타입을 알아내 값을 추가할 수 있다.

```csharp
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```
