# item 58) 전통적인 for문보다 for-each문을 사용하기

## for문의 단점

* 원소에 대한 정보만 필요한데, for문을 사용하면 반복자와 인덱스 변수가 명시되어 코드를 지저분하게 만든다.
* 중첩 for문을 작성할 때, 바깥 반복문에 바깥 원소를 저장하는 변수를 추가해야 한다.

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(suit, j.next()));
```

## for-each문의 장점

* 반복자와 인덱스 변수를 사용하지 않기 때문에 코드가 깔끔하고 오류가 나지 않는다.
* 어떤 컨테이너를 다루는지 신경쓸 필요 없다. (직접 next 메서드, 길이 등을 지정해줄 필요 없다)
* 바깥 원소를 저장할 필요 없이 깔끔하게 구현 가능하다.

```java
for (Suit suit: suits)
    for (Rank rank: ranks)
        deck.add(new Card(suit, rank));
```

* Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다. 따라서 원소들의 묶음을 표현하는 타입을 작성 시, Iterable을 구현해 추후 for-each문을 사용할 수 있도록 하면 좋다.

## for-each문을 사용할 수 없는 상황

1. 파괴적인 필터링 - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 remove 메소드를 호출해야 한다.
2. 변형 - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
3. 병렬 반복 - 여러 컬렉션을 **병렬로 순회**해야 한다면, 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.a
