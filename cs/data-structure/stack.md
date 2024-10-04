---
description: 후입선출을 따르는 자료 구조
---

# Stack

## Stack

* LIFO(Last In First Out), 즉 가장 마지막에 들어온 원소가 가장 처음으로 나가게 되는 자료구조이다.
* FILO(First In Last Out) 라고도 할 수 있다.
* 현실 세계에서는 설거지를 생각해보면 쉽게 이해할 수 있다. 아주 무겁게 쌓인 접시들을 닦아야 하는 상황에서는 가장 위에 있는 접시부터 하나씩 설거지를 해야 한다. 따라서 가장 마지막으로 올려둔 접시가 가장 처음 닦이게 되고, 가장 처음 놓은 접시는 가장 마지막에 닦이게 될 것이다.
* 컴퓨터 세계에서는 함수의 호출 단계를 표현할 때 적합하다. 함수 내부에서 또다른 함수가 호출되는 과정이 반복해서 일어날 때, 보통은 스택에 함수 정보들을 쌓는다. 가장 최근에 호출된 함수가 종료됨에 따라 순차적으로 스택의 정보를 제거한다.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption><p>geeksforgeeks</p></figcaption></figure>

## Stack in Java

### Stack

* Stack 클래스는 레거시인 Vector 클래스를 확장하기 때문에 단일 스레드 환경에서 성능이 떨어진다.
* 또한 Vector에서 제공하는 메서드를 통해 배열의 중간에 데이터를 삽입/삭제하거나 인덱스 기반의 동작을 할 수 있어 LIFO 특성을 강제할 수 없다.

```java
Stack<Integer> stack = new Stack<>();
int idx = stack.search(3);
Integer e = stack.indexOf(1);
```

### ArrayDeque

* 따라서 Stack 클래스 대신 [ArrayDeque](queue.md#arraydeque) 를 Stack으로 사용하는 것이 권장된다.

```java
Deque<String> dq = new ArrayDeque<>();

// 원소를 추가
dq.push("레몬");

// 원소 조회 및 제거
String result = dq.pop();
```
