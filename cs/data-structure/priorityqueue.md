---
description: 우선순위 순서로 꺼낼 수 있는 큐
---

# PriorityQueue

## PriorityQueue

* 원소를 꺼낼 때 우선순위가 높은 원소가 가장 먼저 나오게 되는 큐이다.
* 우선순위 큐는 일반적으로 힙으로 구현된다. 따라서 항상 정렬 상태로 유지된다.
  * 힙은 최댓값을 찾기 위해 사용하는 최대 힙과 최솟값을 찾기 위해 사용하는 최소 힙으로 나뉜다.
* 원소 추가 시 O(lgN), 우선순위가 가장 높은 원소 확인 시 O(1), 우선순위 가장 높은 원소 제거 시 O(lgN)이 소요된다.

## PriorityQueue in Java

<figure><img src="../../.gitbook/assets/image (39) (1).png" alt=""><figcaption><p>geeksforgeeks</p></figcaption></figure>

### PriorityQueue

* JDK 1.5 이후부터 제공되며, AbstractQueue 인터페이스의 구현체이다.
* 직접 정의한 객체를 PriorityQueue에 넣기 위해서는 Comparator를 생성자에 넣어주거나, 객체가 Comparable 인터페이스를 구현하도록 해야 한다. 그렇지 않다면 ClassCastException이 발생한다.
* 작은 순서대로 우선순위가 높기 때문에, 큰 순서대로 우선순위를 하려면, Comparator를 반대로 작성해야 한다.
* offer, poll, remove, add 메서드는 O(logN)이 소요된다.
* null은 들어올 수 없다.
* 계속해서 무한으로 사이즈를 늘릴 수 있는 큐이므로 OutOfMemoryError가 발생할 수 있다.

```java
// 우선순위 큐 생성
PriorityQueue<Integer> priorityQueue = new PriorityQueue<>(); // 작은 숫자가 먼저 나온다.
PriorityQueue<Integer> priorityQueue = new PriorityQueue<>(Collections.reverseOrder()); // 큰 숫자가 먼저 나온다.

// user의 나이가 적은 순서대로 우선순위 큐 생성
PriorityQueue<User> priorityQueue = new PriorityQueue<>((User u1, User u2) -> u1.getAge() > u2.getAge() ? 1 : -1);
// user의 나이가 많은 순서대로 우선순위 큐 생성
PriorityQueue<User> priorityQueue = new PriorityQueue<>((User u1, User u2) -> u1.getAge() < u2.getAge() ? 1 : -1);

// 원소 추가
priorityQueue.offer(3);
priorityQueue.offer(30);

// 우선순위가 가장 높은 원소를 조회하고 삭제
Integer result = priorityQueue.poll();

// 주어진 특정 원소를 삭제
Integer result = priorityQueue.remove(3);

// 우선순위가 가장 높은 원소 조회
Integer result = priorityQueue.peek();

// 우선순위 큐 초기화
priorityQueue.clear();
```

### PriorityBlockingQueue

* 자바 멀티스레딩 환경에서 thread-safe한 형태로 우선순위 큐를 제공한다.
* 내부적으로 lock을 잡고 추가/조회/삭제 작업이 이뤄지기 때문에 thread-safe하다.
* PriorityQueue의 특성을 그대로 가진다.

```java
// 우선순위 큐 생성
PriorityBlockingQueue<String> pbq = new PriorityBlockingQueue<String>();
PriorityBlockingQueue<String> pbq = new PriorityBlockingQueue<String>(List.of("a", "b"));

// 원소 추가
pbq.offer("hello");
pbq.offer("hi");

// 원소 삭제
pbq.remove("hi");

// 우선순위가 가장 높은 원소 조회
String result = pbq.peek();

// 우선순위 큐 원소 개수 조
int size = pbq.size();

// 우선순위 큐 초기화
pbq.clear();
```

