---
description: Key와 Value 형태로 이뤄진 데이터 구조
---

# Map

## 특징

* Key-Value 쌍의 집합을 저장하는 데이터 구조
* 하나의 Key는 하나의 Value와 매핑된다.
* 맵의 구현 방식으로 여러 종류가 있다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## HashMap

* Key-Value 형태로 매핑하여 저장하는 자바의 대표적인 자료구조이다.
* Key 값을 해시 함수를 통해 해싱하여 나온 인덱스에 Value를 저장한다.
* 따라서 일반적인 상황(해시 충돌이 발생하지 않는 상황)에서는 조회 및 삽입에 O(1)이 소요된다.
* Key와 Value에는 null이 들어갈 수 있으므로 주의해야 한다.
* 정렬되지 않은 형태이며 Key는 중복될 수 없지만 Value는 중복되어도 상관 없다.
* Thread-safe하지 않으므로 만약 여러 스레드에서 접근될 수 있다면 ConcurrentHashMap를 사용해야 한다.

### 내부 구현

* Boolean, Number 객체를 대상으로 하는 해시 함수의 경우 해시 충돌이 발생하지 않는 완전한 해시 함수를 만들어 낼 수 있지만, String이나 POJO(Plain old java object)에 대해서는 완전한 해시 함수를 만들어 낼 가능성이 사실상 없다.
* **해시 충돌**이란 서로 다른 값을 해싱했을 때 같은 결과(해시 버킷)가 나오는 경우를 말한다.
* 예를 들어 키 A와 B의 해시 버킷이 같은 상황이고 `A`라는 key의 value에 `Hello` 를 입력한 후 `B`라는 key의 value에 `World`를 입력하면 A의 값의 결과가 덮어쓰여져 문제가 발생할 것이다.
* 해시 충돌을 해결하기 위해 `Open Addressing` 과 `Separate Chaining`이라는 방법이 존재한다.
* **Open Addressing**
  * 데이터를 삽입하려는 해시 버킷이 이미 사용 중인 경우 다른 비어있는 해시 버킷에 데이터를 삽입한다.
  * 데이터를 저장/조회할 해시 버킷을 찾는 방법으로는 충돌 발생 시 다음 해시 버킷으로 1칸씩 이동하는 Linear Probing, 충돌 발생 시 오른쪽으로 1,3,5, ... 칸씩 이동하는 Quadratic Probing, 충돌 발생 시 이동할 칸의 수를 새로운 해시 함수로 계산하는 Double Hashing 방식 등이 존재한다.
  * Linear Probing, Quadratic Probing의 경우 데이터가 연속적으로 저장되기 때문에 Cache hit rate가 높지만, 데이터가 일정 부분에 뭉쳐있는 Clustering 현상이 발생하기 쉬워 다음 빈칸을 찾기 어려워진다는 단점이 있다. Double Hashing 방식은 Clustering 현상이 발생하진 않지만 넓게 분포되어 Cache hit rate가 낮아지게 된다.
  * 데이터를 삭제할 때 효율적인 처리가 어렵고 저장할 데이터가 많아질수록 다음 해시 버킷을 찾는 작업의 소요 시간도 오래걸리게 된다.
* **Separate Chaining**
  * 각 해시 버킷에 LinkedList를 두어 해시 충돌이 일어난다면 해당 리스트에 원소를 추가하는 방식이다.
  * Java HashMap에서는 이 방식을 차용해 특정 데이터 개수 이하면 LinkedList를, 이상이면 Red-Black Tree를 해시 버킷마다 두어 해시 충돌을 해결한다.
    * 데이터가 많아질수록 조회하기 위해 LinkedList를 순회하는 것은 비효율적이므로 Red-Black Tree를 사용하여 성능상 이점을 취할 수 있다.
    * 삽입, 검색, 제거 작업이 O(1) 소요되지만, 두 Key가 동일 인덱스에 매핑되는 해시 충돌이 발생할 경우 LinkedList 혹은 Red-Black Tree로부터 데이터를 조회해야 하므로 성능이 저하될 수 있다.
  * HashMap의 데이터가 일정 개수(load factor \* size) 이상이 되면 해시 버킷의 개수를 두 배로 늘려 해시 충돌을 줄일 수 있다. 다만 이 때 모든 데이터를 읽어 새로운 Seperate Chaining 을 구성해야 한다.
  * 따라서 성능을 높이려면 HashMap 생성 시 적절한 해시 버킷 개수를 지정해야 한다.
*   **보조 해시 함수**

    * 해시 버킷의 개수를 늘릴 때 2배로 확장하기 때문에 해시 버킷의 개수 M이 $$2^a$$형태가 되어 `X.hashcode() % M` 로 인덱스 위치를 계산할 때 일부 비트만을 사용하게 된다. 따라서 해시 함수가 32비트 영역을 고르게 사용하도록 만들더라도 해시 값을 2의 승수로 나누면 해시 충돌이 쉽게 발생할 수 있다.
    * 이를 방지하기 위해 상위 16비트 값을 XOR하는 형태의 보조 해시 함수를 두어 나머지 연산 대신 사용할 수 있도록 한다.

    ```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    ```

### 사용 방법

```java
// 선언 (생성 시 크기를 지정하면 효율적이다)
Map<Integer,String> map = new HashMap<>();
Map<Integer,String> map = new HashMap<>(10);
Map<Integer,String> map = new HashMap<>(10, 0.5f); // load factor를 float 타입으로 지정
Map<Integer,String> map = new HashMap<>(map);
Map<Student,Integer> map = new HashMap<>(Comparator::comparingInt(Student::getScore));

// 추가
map.put(10, "Geeks");
map.putAll(anotherMap);
map.putIfAbsent(10, "Bob");

// 값 변경
map.replace(10, "hello");
map.compute(10, (key, oldValue) -> oldValue + "!");

// 키 또는 값이 존재하는지 확인
map.containsKey(3);
map.containsValue("Ann");

// 키에 해당하는 값 조회
map.get(10);
map.getOrDefault(10, "Bob");

// 키 제거
map.remove(10);

// 크기 조회
int size = map.size();

// 전체 순회 (순회 도중 특정 원소 제거 불가)
for (Map.Entry<Integer, String> e : map.entrySet()) {
  System.out.println(e.getKey() + " " + e.getValue());
}

map.forEach((k, v) -> System.out.println(k + " : " + v));
```

## LinkedHashMap

* HashMap과 동일한 방식으로 저장하면서 내부적으로 Doubly Linked List로 관리되어 원소의 삽입 순서 혹은 접근 순서를 유지하는 특징을 가진다.

```java
// 생성
Map<Integer,String> map = new LinkedHashMap<>();
Map<Integer,String> map = new LinkedHashMap<>(10);
Map<Integer,String> map = new LinkedHashMap<>(10, 0.75f); // load factor를 float 타입으로 지정
Map<Integer,String> map = new LinkedHashMap<>(10, 0.75f, true); // 마지막 액세스 순서를 유지하려면 true, 삽입 순서를 유지하려면 false 지정

// 가장 첫 요소 조회
Map.Entry<Integer, String> entry = map.entrySet().iterator().next();

// 순회
for (Map.Entry<Integer, String> e : map.entrySet()) {
  System.out.println(e.getKey() + " " + e.getValue());
}
```

## TreeMap

* key를 기준으로 정렬되며, NavigableMap 인터페이스를 구현하여 특정 키 구간만 조회하거나, 주어진 키를 기준으로 크거나 작은 Entry를 조회하는 등의 기능도 제공한다.
* 따라서 key의 타입은 반드시 Comparable 인터페이스를 구현하거나 직접 정의한 Comparator가 존재해야 한다.
* 내부적으로 레드블랙트리를 사용하기 때문에 HashMap에 비해 성능이 떨어지지만, 잘 정렬된 데이터를 갖고 있어야 하는 경우 효율적으로 사용할 수 있다.
* key 값을 사용해 value를 조회하는 get 메서드의 경우 시간이 많이 소모되므로 많은 양의 데이터를 가져올 경우 entrySet 메서드를 사용하는 것이 좋다.

```java
// 선언 (생성 시 크기를 지정할 수 없다)
TreeMap<Integer,String> map = new TreeMap<>();
TreeMap<Integer,String> map = new TreeMap<>(map);
TreeMap<Student,Integer> map = new TreeMap<>(Comparator::comparingInt(Student::getScore));

// 추가
map.put(10, "Geeks");
map.putAll(anotherMap);

// 값 변경
map.replace(10, "hello");

// 키 또는 값이 존재하는지 확인
map.containsKey(3);
map.containsValue("Ann");

// 키에 해당하는 값 조회
map.get(10);

// 키 제거
map.remove(10);

// 크기 조회
int size = map.size();

// 전체 순회 (순회 도중 특정 원소 제거 불가)
for (Map.Entry<Integer, String> e : map.entrySet()) {
  System.out.println(e.getKey() + " " + e.getValue());
}

// 부분 맵 조회
int start = 1;
int end = 20;
int mid = (start + end) / 2;

Map headMap = map.headMap(mid); // 첫 원소부터 주어진 키의 원소까지 반환
Map tailMap = map.tailMap(mid); // 주어진 키의 원소부터 마지막 원소까지 반환
Map subMap = map.subMap(start, end); // 주어진 키 범위 내의 원소들을 반환

// 최소, 최대 조회
Map.Entry<Integer, String> entry = map.firstEntry();
Map.Entry<Integer, String> entry = map.lastEntry();
Integer minKey = map.firstKey(); // 가장 작은 키 반환
Integer maxKey = map.lastKey(); // 가장 큰 키 반환
```
