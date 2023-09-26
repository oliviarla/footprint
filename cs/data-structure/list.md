# List

## 특징

* 선형 자료구조를 구현할 때 사용한다.
* 일반적으로 리스트라고 하면, LinkedList를 말한다.
* 배열과 다르게 메모리에 원소들을 연속적으로 할당하지 않는다.
* 따라서 크기가 가변적이며, 공간 지역성의 장점이 없어 Cache Hit Rate가 낮다.
* 원소에 접근할 때는 O(N)의 시간 복잡도를 갖는다.
* 삽입과 삭제의 경우 O(N)의 시간 복잡도를 갖는다.

## ArrayList

* Array 특징과 List 특징을 합친 자료구조
* 확장 가능하거나 크기 조정 가능한 배열을 기반으로 생성된다.
* &#x20;인덱스 기반의 데이터 구조이기 때문에 특정 위치의 데이터 탐색이 빠르다.
* Array의 경우 primitive 타입을 담을 수 있지만, ArrayList는 primitive 타입은 담을 수 없다.
* 삽입과 삭제 시 뒤 원소를 모두 하나씩 밀거나 땡긴다. 따라서 삽입 및 삭제 작업이 빈번한 상황에는 부적합하다.
* Array의 특징이 있기 때문에 요소는 인접한 위치에 저장된다.
* 중복 저장이 가능하다.
* Null 값을 입력할 수 있다.
* 공간이 부족한 경우 현재 **공간의 절반** 만큼의 메모리를 더 할당받는다.

```java
// 선언
List<String> lst = new ArrayList<>(); // 기본 선언
List<String> lst = new ArrayList<>(100); // 초기 크기 설정

// 추가
lst.add("hello");
lst.add(0, "hello"); // 특정 index에 원소 추가
lst.addAll(List.of("hello world!", "hello guys!")); // 컬렉션의 모든 원소 추가

// 인덱스 기반 조회
int idx = 0;
String s = lst.get(idx);

// 리스트 길이 조회
int size = lst.size();

// 제거 시 index/객체 모두 입력 가능
// +) List<Integer>의 경우 index 기반 제거가 먼저 동작한다.
lst.remove(1);
lst.remove("hello");

// 검색
boolean isContainNumber1 = lst.contains("hello"); // 특정 원소 포함하는지 확인
int indexOfNumber1 = lst.indexOf("hello"); // index 반환, 탐색 실패 시 -1 반환
```

## LinkedList

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* 데이터가 컴퓨터 메모리 내부에 순차적으로 저장되지 않고, **주소 단위로 서로 연결**되는 선형 데이터 구조이다.
* 삽입과 삭제는 단순히 Prev와 Next 값을 변경해주면 되기 때문에 O(1)의 시간 복잡도를 갖는다.
* 특정 인덱스의 원소를 탐색하려면 O(N)의 시간 복잡도를 갖는다.
  * 반복자를 통해 순회해야만 특정 위치의 원소를 조회할 수 있다.

```java
// 선언
List<String> lst = new LinkedList<>();

// 추가
lst.add("A");
lst.add(2, "E"); // 특정 인덱스에 추가
lst.addAll(List.of("hello world!", "hello guys!")); // 컬렉션의 모든 원소 추가

// 제거
lst.remove(); // 0번째 index제거
lst.remove("B"); // 특정 원소를 찾아 제거
lst.remove(3); // index로 제거
lst.clear(); //모든 원소 제거
```

> 자바의 LinkedList 클래스는 Deque 인터페이스까지 구현하고 있지만 여기서는 다루지 않도록 한다.

## Vector

* legacy class 이므로 자주 사용되지 않는다.
* 동기화를 보장해 공유 자원이나 복수 사용자 존재 시 안전하다고는 하지만, Concurrent 패키지의 클래스 사용을 권장한다.
* 하나의 스레드만 자원을 이용하는 경우 성능 저하가 발생할 수 있다.
* 공간이 부족한 경우 현재 **공간**을 두 배로 늘리기 때문에, 점점 메모리를 많이 사용하게 된다.
* 사용법은 ArrayList와 동일하다.
