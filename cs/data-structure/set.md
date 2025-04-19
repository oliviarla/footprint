---
description: 중복을 허용하지 않는 데이터 셋
---

# Set

## 특징

* 고유한 원소들을 저장하는 자료구조
* 수학의 집합 개념과 유사하다.
* 중복된 값을 저장할 수 없다.
* 일반적으로 저장되는 순서를 유지할 수 없다. 따라서 리스트와 달리 index 기반 탐색이 불가능하다.
* 두 가지 구현 방식이 있다.
  *   Hash-Based Set

      * 해시 테이블로 표현되는 Set이다.
      * 각 원소를 해싱하여 얻은 인덱스 위치에 원소가 저장된다.
      * 다음 그림처럼 여러 원소(Key)들을 Hash Table에 저장하는 방식이다.

      <figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
  * Tree-Based Set
    * 자가 균형 이진 탐색 트리로 표현되는 Set이다.
    * 일반적으로 red-black tree를 사용한다.
    * 각 원소는 트리의 노드에 저장된다.

## HashSet

* 해시 테이블을 두고, 원소를 해싱하여 만든 인덱스에 저장하는 방식
* 단순히 원소를 해싱하여 해당 위치로 이동하면 되기 때문에 탐색, 추가, 제거 작업은 O(1)이 소요된다.
* 자바의 HashSet은 아래와 같이 HashMap을 이용하여 구현되어 있다.
* HashMap의 Key에는 Set의 요소들을 가지며, Value에는 Dummy Object(쓸모없는 임시 값)을 가진다.

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable
{
    // ...
    private transient HashMap<E,Object> map;
    // ...
}
```

```java
// 선언
Set<Integer> set = new HashSet<>();
Set<Integer> set = new HashSet<>(10); //초기 크기를 지정

// 추가
set.add(2);
set.addAll(anotherSet);

// 제거
set.remove(2); // 특정 값 제거 (인덱스 기반 제거 불가능)
set.clear(); // 모든 원소 제거

// 길이
int length = set.size();

// 포함 여부 확인
boolean isContain4 = set.contains(4);
```

## TreeSet

* 자가 균형 이진 트리(self balancing binary search tree)를 사용하기 때문에 데이터의 추가 및 삭제, 탐색 작업에 O(log(N))이 소요된다.
* 자바의 TreeSet은 레드 블랙 트리를 사용해 구현된 TreeMap을 이용해 구현되어 있다.
* 정렬이 가능하여 최소/최대값을 찾거나 특정 값을 기준으로 크거나 작은 값들을 빠르게 찾을 수 있다.

```java
// 선언
TreeSet<Integer> set = new TreeSet<>();
TreeSet<Integer> set = new TreeSet<>(comparator); // Comparable 인터페이스를 구현하지 않았다면 명시해주어야 한다.

// 추가
set.add(2);
set.addAll(anotherSet);

// 제거
set.remove(2); // 특정 값 제거 (인덱스 기반 제거 불가능)
set.clear(); // 모든 원소 제거

// 길이
int length = set.size();

// 포함 여부 확인
boolean isContain4 = set.contains(4);

// 비교 
// +) Set 인터페이스에는 없는 메서드들이므로, Set이 아닌 SortedSet 또는 TreeSet의 객체여야 동작한다.
Integer minOfSet = set.first();
Integer maxOfSet = set.last();
Integer moreThan3 = set.higher(3);
Integer lessThan3 = set.lower(3);
```

## LinkedHashSet

* HashSet을 상속받았으며, doubly-linked list를 기반으로 하는 LinkedHashMap을 통해 구현되어 있다.
* 일반적인 Set과 다르게 **입력된 순서를 유지**한다.

```java
// 선언
Set<Integer> set = new LinkedHashSet<>();
Set<Integer> set = new LinkedHashSet<>(100, .5f); // 최대 용량 및 용량 초과 시 늘리는 비율을 지정 가능

// 추가
set.add(2);
set.addAll(anotherSet);

// 제거
set.remove(2); // 특정 값 제거 (인덱스 기반 제거 불가능)
set.clear(); // 모든 원소 제거

// 길이
int length = set.size();

// 포함 여부 확인
boolean isContain4 = set.contains(4);
```
