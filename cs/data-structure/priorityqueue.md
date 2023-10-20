---
description: 우선순위 순서로 꺼낼 수 있는 큐
---

# PriorityQueue

## PriorityQueue

* 원소를 꺼낼 때 우선순위가 높은 원소가 가장 먼저 나오게 되는 큐이다.
* 우선순위 큐는 일반적으로 힙으로 구현된다.
  * 최댓값을 찾기 위해 사용하는 힙: 최대 힙
  * 최솟값을 찾기 위해 사용하는 힙: 최소 힙
* 원소 추가 시 O(lgN), 우선순위가 가장 높은 원소 확인 시 O(1), 우선순위 가장 높은 원소 제거 시 O(lgN)이 소요된다.

## PriorityQueue in Java

### Priority Queue

```java
// 우선순위 큐 생성
PriorityQueue<Integer> priorityQueue = new PriorityQueue<>(); // 작은 숫자가 먼저 나온다.
PriorityQueue<Integer> priorityQueue = new PriorityQueue<>(Collections.reverseOrder()); // 큰 숫자가 먼저 나온다.

// 값 추가
priorityQueue.offer(3);
priorityQueue.offer(30);

// 우선순위가 가장 높은 원소를 조회하고 삭제
Integer result = priorityQueue.poll();

// 특정 원소를 삭제
Integer result = priorityQueue.remove(3);

// 우선순위가 가장 높은 원소를 조회
Integer result = priorityQueue.peek();

// 우선순위 큐 초기화
priorityQueue.clear();
```

### PriorityBlockingQueue



