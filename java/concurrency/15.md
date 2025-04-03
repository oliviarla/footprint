# 15장

## 들어가며

* java.util.concurrent 패키지의 Semaphore, ConcurrentLinkedQueue등은 단순히 synchronized 구문으로 동기화를 맞춰 사용하는 것보다 속도가 빠르고 확장성이 좋다.
* 그 이유는 단일 연산 변수와 대기 상태에 들어가지 않는 논블로킹 동기화 기법 덕분이다.
* 멀티 스레드 동작 환경에서 데이터의 안정성을 보장하는 방법으로 락을 사용하는 대신 저수준의 하드웨어에서 제공하는 CAS 명령을 사용한다.
* 여러 스레드가 동일한 데이터에 대해 경쟁하면서 대기 상태에 들어가지 않으므로 스케줄링 부하를 줄여주고 데드락, 기타 문제가 발생할 위험이 없다.
* 락 기반이라면 특정 스레드가 락을 확보해놓고 sleep 하거나 반복문을 실행하면 다른 스레드가 계속 대기해야 하는 문제가 있다. 또한 대기 상태였던 스레드를 다시 실행하고자 할 때 CPU 스케줄을 넘겨받기 위해 기다려야 할 수도 있다. 이와 같이 스레드의 실행과 중단 과정이 반복되면 CPU에 상당한 부하를 발생시키며 락을 기반으로 작업하는 클래스는 경쟁이 심해질수록 실제 작업 처리 시간보다 동기화 처리 시간이 더 많이 걸릴 수 있다.
* volatile 변수는 컨텍스트 스위칭, 스레드 스케줄링과 관련이 없으므로 락에 비해 훨씬 가볍다. 조회에 대해서는 락과 비슷하지만 복합 연산을 하나의 단일 연산으로 처리하는 기능을 제공하진 않는다. 다른 변수와 연관된 상태(ex. 이전값에 1을 더하는 것(++i)은 `현재 값 조회 → 값에 1을 더하기 → 새로운 값 덮어쓰기`를 단일 연산으로 처리해야 한다)를 다뤄야 한다면 volatile 변수를 사용할 수 없다. 따라서 counter, mutex는 volatile 변수를 통해 구현할 경우 전혀 성능 상 이득을 볼 수 없다.
* 우선 순위 역전: 락을 확보하였고 지연되고 있는 스레드의 우선 순위가 락을 대기하고 있는 스레드보다 낮아질 경우 프로그램 성능에 심각한 영향이 갈 수 있다.

## 단일 연산 변수

* 단일 연산 변수(Atomic Variable)는 volatile 변수와 동일한 메모리 유형을 가지며 단일 연산으로 값을 변경할 수 있다.
* 세밀하고 단순한 작업을 처리하는 경우 락 대신 **일단 값을 변경하고 다른 스레드 간섭이 없다면 값을 변경하는 것**이다. 다른 스레드의 간섭이 있었다면 해당 연산은 실패하고 재시도를 할 수 있다.
* 멀티프로세서 연산을 고려해 만들어진 프로세서는 공유된 변수에 동시 작업이 필요한 상황을 지원하기 위해 별도의 명령어를 제공하였다.
  * 초기 프로세서는 test-and-set / fetch-and-increment / swap 등의 단일 연산을 제공했다.
  * 최근에는 compare-and-swap / load-linked / store-conditional 등의 단일 연산을 제공한다.

### Compare and Swap

* 작업 대상 메모리 위치(V), 예상하는 기존 값(A), 새로 설정할 값(B) 를 인자로 넘겨 V 위치의 값이 A일 경우 B로 변경하는 단일 연산이다.
* 값 변경의 성공 실패에 관계 없이 현재 V에 있는 값을 반환한다. 만약 다른 스레드가 값을 변경했다면 그것을 확인하기 위함이다.
* 다음은 자바 언어 레벨에서 구현해 본 CAS 코드이다.

```java
@ThreadSafe
public class SimulatedCAS {
    @GuardedBy("this")
    private int value;
    
    public synchronized int get() {
        return value;
    }
    
    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue = value;
        
        if (oldValue == expectedValue) {
            value = newValue;
        }
        
        return oldValue;
    }
    
    public synchronized boolean compareAndSet(int expectedValue, int newValue) {
        return (expectedValue == compareAndSwap(expectedValue, newValue));
    }
}

@ThreadSafe
public class CasCounter {
    private SimulatedCAS value;

    public SimulatedCAS getValue() {
        return value;
    }
    
    public int increment() {
        int v;
        
        do {
            v = value.get();
        } while (v != value.compareAndSwap(v, v + 1));
        
        return v + 1;
    }
}
```

* 락을 사용하면 JVM과 운영체제 내부에서 복잡한 절차를 통해 처리된다. CAS 연산의 경우 JVM에서 실행해야 할 특별한 루틴도 없고 운영체제 내부에서 함수 호출, 스케줄링 작업을 할 필요도 없다.
* 락에 비해 CAS가 가지는 가장 큰 단점은 호출하는 프로그램에서 직접 재시도를 하는 등 스레드 경쟁 조건에 대한 처리를 해주어야 한다는 것이다.
* CPU가 하나면 CPU 간 동기화 작업이 필요 없으므로 몇 번의CPU 사이클으로 완료된다. 책이 쓰인 시점을 기준으로 다중 CPU 시스템이라면 약 10\~150번 CPU 사이클으로 완료된다고 한다.
* JVM은 하드웨어 프로세서에서 CAS 연산을 지원하는 경우 네이티브코드를 통해 호출한다. 하드웨어에서 CAS 연산을 지원하지 않는다면 스핀락을 이용해 CAS 연산을 구현한다. 이와 같은 저수준의 CAS 연산은 AtomicXXX 클래스를 통해 제공된다.

### 단일 연산 변수 클래스(AtomicXXX)

* 스레드가 경쟁하는 범위를 하나의 변수로 좁혀주는 효과가 있으며 대부분의 경우 락을 사용해 구현된 것보다 빠르다.
* 내부의 스레드가 지연되는 현상이 거의 없으며 스레드 간 경쟁이 발생해도 더 쉽고 빠르게 경쟁 상황을 처리할 수 있다.
* volatile 변수의 기능 + 읽고 변경하고 쓰는 단일 연산 기능을 합쳐서 지원한다.
* 단순한 get, set 메서드는 volatile 변수와 동일한 기능을 한다.
* compareAndSet 메서드 등 단일 연산 기능을 제공한다.
* AtomicInteger 클래스는 Counter 클래스와 달리 동기화를 위한 하드웨어 기능을 직접적으로 이용한다. 따라서 경쟁이 발생하는 상황에서 높은 확장성을 제공한다.
* 여러 값을 가지는 변경 불가능한 변수에 대한 참조를 단일 연산으로 변경하고자 할 때는 아래와 같이 AtomicReference에 여러 값을 클래스 형태로 래핑해 넣어두면 된다.

```java
public class CasNumberRange {
    @Immutable
    private static class IntPair {
        final int lower; // 불변조건 : lower <= upper
        final int upper;

        public IntPair(int lower, int upper) {
            this.lower = lower;
            this.upper = upper;
        }
    }

    private final AtomicReference<IntPair> values = new AtomicReference<IntPair>(new IntPair(0, 0));

    public int getLower() {
        return values.get().lower;
    }

    public int getUpper() {
        return values.get().upper;
    }

    public void setLower(int i) {
        while (true) {
            IntPair oldv = values.get();
            if (i > oldv.upper) {
                throw new IllegalArgumentException("Can't set lower to " + i + " > upper");
            }
            IntPair newv = new IntPair(i, oldv.upper);
            if (values.compareAndSet(oldv, newv)) {
                return;
            }
        }
    }
}
```

* 락이든 단일 연산 변수든 결국은 스레드 간 데이터를 공유하지 않는 것이 가장 성능이 좋긴 하다.

## 넌블로킹 알고리즘

* 특정 스레드에서 작업이 실패하거나 대기 상태에 들어가는 경우가 발생해도 다른 스레드들이 실패하거나 대기 상태에 들어가지 않는 알고리즘
* 각 작업단계마다 일부 스레드는 항상 작업을 진행할 수 있는 경우 락 프리 알고리즘이라고 한다.
* 데드락이나 우선 순위 역전 등의 문제가 발생하지 않는다. 물론 지속적으로 재시도를 한다면 라이브락 등의 문제가 발생할 수 있다.
* 넌블로킹 알고리즘 구현 시 가장 핵심이 되는 부분은 데이터의 일관성을 유지하면서 단일 연산 변경 작업의 범위를 하나의 변수로 제한하는 것이다.
* ConcurrentStack 클래스의 경우 Node 클래스로 구성된 연결 리스트로 구현되며, top 변수를 두어 push 메서드를 통해 새로 들어온 노드의 next 필드에 기존의 top 변수를 저장하고, 새로운 노드는 스택의 top변수에 대입한다. 이 작업은 CAS 연산에 의해 수행되며 기존 top 변수가 변경되지 않았다면 성공한다. 만약 다른 스레드에 의해 top 변수가 변경되었다면 새로운 top 변수를 기준으로 재시도해야 한다.
* 대기 상태에 들어가지 않는 논블로킹 알고리즘은 volatile 쓰기 특성이 있는 compareAndSet 연산을 통해 단일 연산 특성과 가시성을 보장하므로 thread-safe하다.

## Linked Queue

* Linked Queue의 경우 head와 tail에 직접적으로 접근 가능해야 하므로 스택보다 복잡하다. head, tail에 대한 참조를 필드에 저장해두고, 새로운 항목을 큐에 추가하려면 tail에 대한 두 개의 참조(큐 내부에서 직접 접근을 위한 필드와 마지막 노드가 가지는 next 필드)가 단일 연산으로 함께 변경되어야 한다.
* 하지만 두 개의 변수를 단일 연산 변수로 업데이트할 수 없다. 두 번의 CAS 연산을 사용하는 수밖에 없는데, 각 CAS 연산들이 항상 둘 다 성공하지 않을 수 있으므로 안전하지 않다.
* **데이터 구조가 여러 단계의 변경 작업을 거치는 과정을 포함하여 일관적인 상태로 유지**되어야 한다.
  * 스레드 B는 스레드 A가 값을 변경하고 있을 때 변경 작업이 진행중임을 알고, 스레드 B가 하려는 변경 작업을 바로 시작하지 않도록 해야 한다.
* 특정 스레드에서 오류가 발생했을 때 다음 스레드가 큐의 데이터를 사용하지 못하는 상황이 발생하지 않도록 해야 한다. 스레드 A가 처리 중인 작업을 마쳐야 스레드 B가 데이터 구조를 접근해 사용가능하다는 것에 대한 정보를 데이터 구조에 넣어둔다.
* 스레드 A가 해야 할 마무리 작업을 스레드 B가 도와주면 스레드 A가 작업을 마칠 때 까지 기다리는 대신 직접 작업을 수행한 후 자신 스레드가 해야할 일을 이어서 수행할 수 있다.

## 마이클 스콧 알고리즘

* head, tail 필드는 비어있는 노드를 참조한다. tail 노드는 항상 비어있는 노드, 큐의 마지막 항목을 참조하며, 변경이 진행중일 때에는 맨 뒤에서 두번째 항목을 가리킨다.
* 새로운 노드를 추가할 땐 두 개의 참조를 변경해야 한다.
  * 현재 큐의 마지막 노드가 가지는 next 참조 값을 새로운 노드로 변경한다.
  * 꼬리를 가리키는 변수가 새로 추가된 노드를 가리키도록 참조를 변경해야 한다.
* 큐에 변경 요청이 들어오지 않은 평온한 상태라면 tail 필드가 가리키는 노드의 next 참조 값이 null이어야 하고, 변경 중인 상태라면 null이 아니어야 한다. 어느 스레드든 tail 필드가 가리키는 노드의 next 참조 값이 null일 경우에만 다음 항목을 가리키도록 하여 항상 원하는 작업을 마치도록 할 수 있다.
* 작업 순서
  1. Linked Queue에 데이터를 추가하기 위해 먼저 해당하는 큐가 변경중인지 확인한다.
  2. 만약 tail 필드가 가리키는 노드의 next 참조 값이 null이 아닌 다른 노드라면, tail 변수의 참조를 다음 항목으로 넘겨주는 작업을 대신 처리한다.
  3. 이를 반복하여 큐가 평온한 상태가 되도록 만든다.
  4. 원래 자신이 넣고자 했던 노드를 추가한다.
* 새로 추가하는 노드를 맨 마지막 노드의 next 참조로 연결시킬 때에는 CAS 연산을 사용한다. 만약 두 개 이상의 스레드가 동시에 큐의 끝에 새로운 노드를 연결하려 하면 실패하는 스레드들이 발생할 것이다. 실패하더라도 tail 변수의 참조 값을 다시 읽어 재시도하면 된다.
* tail 필드가 가리키는 참조를 맨 마지막 노드로 변경하는 작업에도 CAS 연산을 사용한다. 이 작업은 앞서 다뤘듯 작업중인 스레드가 아닌 다른 스레드들이 도와줄 수 있다. 만약 도와주는 과정에서 일부 스레드가 실패하더라도 결국은 tail 필드 최신화 작업이 성공한 것이기 때문에 재시도할 필요는 없다.
*   ConcurrentLinkedQueue의 경우 각 노드 객체를 volatile로 저장해두고 사용한다. 큐의 노드 같이 자주 생성하면서 오래 사용하지 않는 객체 타입으로 AtomicReference로 두면 매번 생성하고 없애는 부하가 생긴다.

    * JDK 8 이전에는 노드 간 연결 구조를 변경할 때에는 AtomicReferenceFieldUpdater 클래스를 사용했다. 이 클래스는 현재 사용중인 volatile 변수에 대한 뷰를 리플렉션 기반으로 나타내어 volatile 변수에 대해 CAS 연산을 사용할 수 있도록 해준다.
    * newUpdater 팩토리 메서드를 사용해 생성할 수 있으며, 지정한 클래스의 모든 인스턴스에 들어있는 변수 값을 변경할 때 사용할 수 있다.
    * 일반적인 단일 연산 변수(AtomicXXX)에 비해 보장되는 연산의 단일성이 약하다. 업데이터를 통하지 않고 직접 변수가 변경된다면 연산의 단일성을 보장할 수 없다. 따라서 모든 스레드에서 해당 변수를 변경할 때 compareAndSet 등 단일 연산 메서드를 사용해야만 한다.

    ```java
    private static class Node<E> {
      private volatile E item;
      private volatile Node<E> next;

      private static final
          AtomicReferenceFieldUpdater<Node, Node>
          nextUpdater =
          AtomicReferenceFieldUpdater.newUpdater
          (Node.class, Node.class, "next");
      // ...
    ```

    * JDK 8 이후부터는 아래와 같이 UNSAFE 정적 메서드를 사용해 native 메서드를 직접 호출한다.

    ```java
    private static class Node<E> {
        volatile E item;
        volatile Node<E> next;

        /**
         * Constructs a new node.  Uses relaxed write because item can
         * only be seen after publication via casNext.
         */
        Node(E item) {
            UNSAFE.putObject(this, itemOffset, item);
        }

        boolean casItem(E cmp, E val) {
            return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
        }

        void lazySetNext(Node<E> val) {
            UNSAFE.putOrderedObject(this, nextOffset, val);
        }

        boolean casNext(Node<E> cmp, Node<E> val) {
            return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
        }

        // Unsafe mechanics

        private static final sun.misc.Unsafe UNSAFE;
        private static final long itemOffset;
        private static final long nextOffset;
        // ...
    }
    ```

## ABA 문제

* GC가 없는 환경에서 노드를 재사용하는 알고리즘을 구현할 때 CAS 연산을 사용하다보면 발생할 수 있는 문제이다.
* 변수의 값이 A였다가 B로 바뀌었다가 다시 A로 바뀌는 경우도 변경사항이 있었다는 것으로 간주하고 재시도해야 하는 경우가 있다. 참조 값 하나만 변경하는 대신 참조와 버전 번호를 한꺼번에 변경하도록 하면 이런 상황을 처리할 수 있다.
* AtomicStampedReference 클래스는 두 개의 값에 대한 조건부 단일 연산 업데이트 기능을 제공한다. 객체에 대한 참조와 숫자 값을 함께 변경할 수 있으므로, 버전 번호를 사용할 수 있다.
* AtomicMarkableReference 클래스는 객체에 대한 참조와 bool 값을 함께 변경할 수 있어 노드 객체를 그대로 두고 삭제 여부 필드만 변경하는 경우에 유용하게 사용할 수 있다.
