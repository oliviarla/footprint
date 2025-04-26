---
description: 선입선출을 기반으로 한 선형 자료구조
---

# Queue

## Queue

* FIFO(First In First Out), 즉 선입선출을 기반으로 하는 선형 자료 구조이다.
* 큐에서 데이터를 꺼낼 때(dequeue)에는 큐의 가장 앞에 저장된 (가장 먼저 입력된) 값이 꺼내지며, 데이터를 집어 넣을 때(enqueue)에는 큐의 가장 끝에 저장된다.
* 큐는 기본적으로 배열 혹은 리스트를 사용해 구현되며, 배열 양 끝의 값만 접근이 가능하다.
* 큐의 가장 앞부분을 보통 head라고 부르며, 뒷부분은 보통 tail, rear라고 부른다.

<figure><img src="../../.gitbook/assets/image (33) (1).png" alt=""><figcaption><p>geeksforgeeks</p></figcaption></figure>

## Deque

* Doubly-Ended Queue
* 큐의 양쪽에 데이터를 넣고 뺄 수 있는 자료구조
* 자바에서 Deque의 구현체는 LinkedList, ArrayDeque, LinkedBlockingDeque, ConcurrentLinkedDeque가 있다.
* 원소를 삽입하는 것을 `offer`라고 하며, 원소를 꺼내 제거하는 것을 `poll`이라고 한다. 현재 큐의 끝에 저장된 값을 확인하는 것을 `peek`이라고 한다.

## Queue in Java

<figure><img src="../../.gitbook/assets/image (38) (1).png" alt=""><figcaption><p>Java에서 제공하는 Queue 클래스 구조 (geeksforgeeks)</p></figcaption></figure>

### LinkedList

* ArrayList는 가장 첫 번째 데이터를 제거할 경우, 빈 공간이 생기게 되고 이를 채우기 위해 뒷 원소들을 모두 앞으로 이동시켜야 하므로 비효율적이다.
* 따라서 Java에서는 LinkedList를 기반으로 Queue를 제공한다.
* 큐에 원소를 추가할 때에는 offer 메서드를 사용하는게 좋다.
  * add 메서드의 경우 용량 초과로 삽입이 불가능하면 IllegalStateException을 발생시킨다.
  * offer 메서드의 경우 용량 초과로 삽입이 불가능하면 false를 반환한다.
* 큐의 원소를 조회한 후 삭제할 때에는 poll 메서드를 사용하는게 좋다.
  * remove 메서드의 경우 큐가 비어있으면 NoSuchElementException을 발생시킨다.
  * poll 메서드의 경우 큐가 비어있으면 null을 반환한다.
* 큐의 원소를 조회만 할 때에는 peek 메서드를 사용하는게 좋다.
  * element 메서드의 경우 큐가 비어있으면 NoSuchElementException을 발생시킨다.
  * peek 메서드의 경우 큐가 비어있으면 null을 반환한다.

```java
Queue<String> q = new LinkedList<>();

// rear에 원소를 추가
boolean succeed = q.add("레몬");
boolean succeed = q.offer("오렌지"); 

// head의 원소를 조회하고 삭제
String result = q.poll();
String result = q.remove();

// head의 원소를 조회
String result = q.peek();
String result = q.element();
```

### ArrayDeque

* ArrayDeque는 말그대로 **배열 기반**으로 구현되어 head, tail 포인터를 가지는 **Deque**이다.
* ArrayDeque의 클래스 구조는 아래와 같다.
* Stack이나 Queue로 사용할 수 있다.
  * Stack 클래스나 LinkedList 클래스를 Queue로 사용할 때 보다 빠른 성능을 보인다.

<figure><img src="../../.gitbook/assets/image (34) (1).png" alt=""><figcaption></figcaption></figure>

* 기본 생성자 사용 시, 초기 배열 크기는 16이다.
* LinkedList에 원소를 추가하는 방식과 다르게, ArrayDeque는 원소 추가 시 배열이 꽉 찼더라도 자체적으로 크기를 늘리기 때문에 ~~이론상 무한대로~~ 추가가 가능하다.

#### Dueue로 사용하기

```java
Deque<String> dq = new ArrayDeque<>();

// 원소를 추가
// 배열 크기가 꽉차도 사이즈를 키우면서 추가하므로, addFirst, addLast사용해도 예외가 발생하지 않으므로 괜찮음
dq.addFirst("레몬");
dq.addLast("사과");
boolean succeed = q.addAll(List.of("아보카도", "키위"));
boolean succeed = dq.offerFirst("오렌지");
boolean succeed = dq.offerLast("포도"); 

// head의 원소를 조회하고 삭제
// removeFirst, removeLast 메서드도 사용가능하지만, 예외를 발생시키기 때문에 사용 자제
String result = dq.pollFirst();
String result = dq.pollLast();

// head의 원소를 조회
// getFirst, getLast 메서드도 사용가능하지만, 예외를 발생시키기 때문에 사용 자제
String result = q.peekFirst();
String result = q.peekLast();
```

#### Queue로 사용하기

```java
Queue<String> q = new ArrayDeque<>();

// 원소를 추가
boolean succeed = q.offer("오렌지");

// head의 원소를 조회하고 삭제
String result = q.poll();

// head의 원소를 조회
String result = q.peek();
```
