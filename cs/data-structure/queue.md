# Queue

## Queue

* FIFO(First In First Out), 즉 선입선출을 기반으로 하는 선형 자료 구조이다.
* 큐에서 데이터를 꺼낼 때(dequeue)에는 큐의 가장 앞에 저장된 (가장 먼저 입력된) 값이 꺼내지며, 데이터를 집어 넣을 때(enqueue)에는 큐의 가장 끝에 저장된다.

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption><p>geeksforgeeks</p></figcaption></figure>





## Deque

* Doubly-Ended Queue
* 큐의 양쪽에 데이터를 넣고 뺄 수 있는 자료구조
* 자바에서 Deque의 구현체는 LinkedList, ArrayDeque, LinkedBlockingDeque, ConcurrentLinkedDeque가 있다.
* 원소를 삽입하는 것을 offer라고 하며, 원소를 꺼내 제거하는 것을 poll이라고 한다. 현재 큐의 끝에 저장된 값을 확인하는 것을 peek이라고 한다.
