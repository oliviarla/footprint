# item 81) wait와 notify보다는 동시성 유틸리티를 애용하라Page 3

## wait, notify

* **스레드의 상태 제어**를 위한 메소드
* wait()는 가지고 있던 고유 락을 해제하고, 스레드를 잠들게 하는 역할을 하는 메서드이다.
* notify()는 잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 역할을 하는 메서드이다.
* wait와 notify는 올바르게 사용하기 까다로우니 고수준의 동시성 유틸리티를 사용하자.

## 동시성 유틸리티

* java.util.concurrent 동시성 유틸리티는 **실행자 프레임워크**(item80)**, 동시성 컬렉션, 동기화 장치** 로 나눌 수 있다.

## 동시성 컬렉션

* List, Queue, Map 등 표준 컬렉션 인터페이스에 동시성을 추가해 구현한 고성능 컬렉션이다. 동기화를 내부에서 수행하여 높은 동시성에 도달할 수 있다.
* 동시성을 무력화하는 것은 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

#### **상태 의존적 수정 메서드**

* 여러 기본 동작을 하나의 원자적 동작으로 묶는 메서드
* Java 8 이후, 일반 컬렉션 인터페이스에 디폴트 메서드 형태로 추가되어 있다.
* putIfAbsent(key, value)
* 주어진 key에 매핑된 value가 없을 때에만 새 값을 put한다. 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환한다.
* 아래 예제를 보면, ConcurrentHashMap은 검색 기능에 최적화 되어있기 때문에 get() 메서드 수행 후 putIfAbsent 사용하면 빠르게 동작한다.

```tsx
private static final ConcurrentMap<String, String> map =
        new ConcurrentHashMap<>();

public static String intern(String s) {
    String result = map.get(s);

    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```

**동기화된 컬렉션**

* 동시성 컬렉션을 사용하면 훨씬 좋으므로 낡은 기술이다.
* synchronizedXXX 와 같은 컬렉션이다.

**작업 완료를 기다리는 인터페이스**

* 이외에도 일부 컬렉션 인터페이스는 작업이 성공적으로 완료될 때까지 기다리도록 설계되었다.
* 예를 들어, BlockingQueue는 ThreadPoolExecutor 등 **대부분의 실행자 서비스 구현체에서 작업 큐(생산자-소비자 큐)로 사용**된다.
* 생산자는 작업을 큐에 추가하고, 소비자가 해당 작업을 꺼내 처리하는 형태

## 동기화 장치

* 스레드가 다른 스레드를 대기할 수 있게 하여 작업의 조율을 돕는다.
* 종류: CountDownLatch, Semaphore, Phaser(가장 강력한 동기화 장치), CyclicBarrier, Exchanger

### **CountDownLatch**

* 일회성 장벽 역할을 한다.
* 하나 이상의 스레드가 또다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
* 생성자로 int 값을 받으며, 이 값이 countDown 메서드를 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정한다.
* 아래 예제는 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 메서드이다.
* 매개변수로는 실행할 실행자, 동시에 수행 가능한 동작 개수(동시성 수준)을 받는다.

```csharp
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비가 됐음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알린다.
                    done.countDown();
                }
            });
        }

        ready.await();// 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown();// 작업자들을 깨운다.
        done.await();// 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
```

* for문 아래 로직을 보면 쉽게 이해할 수 있다.
  * ready 래치: 작업자 스레드들이 준비 완료됨을 타이머 스레드에 통지
  * start 래치: 모든 작업자가 준비되는 것(ready.await() 완료)을 기다린 후, 작업자들이 action.run()으로 작업 실행할 수 있게 countDown한다.
  * done 래치: 모든 작업자가 작업을 마치는 것을 기다린다.
* for문으로 실행자가 concurrency 개수 만큼 스레드를 생성한다. 따라서, 스레드 기아 교착상태가 발생하지 않으려면 실행자의 생성 가능한 스레드 개수는 concurrency 개수 이상이어야 한다.
* InterruptedException 발생 시 interrupt()를 호출하면, 해당 스레드가 하던 작업을 멈추고 실행자에게 인터럽트 발생 여부를 알리면 실행자가 인터럽트를 적절히 처리할 수 있다.

> System.nanoTime: System.currentTimeMillis보다 System.nanoTime가 더 정확하고 정밀하다. 또한, 시스템의 실시간 시계의 시간 보정에 영향받지 않는다.

## 레거시 wait, notify 다루기

### wait 메서드

* 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용
* Lock 객체의 wait 메서드는 반드시 동기화 영역 안에서 호출해야 한다.
* 아래 예시처럼 반드시 대기 반복문 관용구를 사용하고, 반복문 밖에서는 호출하지 말 것

```less
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait();// 락을 놓고, 깨어나면 다시 잡는다.
    }

    ...// 조건이 충족됐을 때의 동작을 수행한다.
}
```

* 조건이 충족되어 notify 메서드 호출 후 다시 wait 메서드로 대기상태로 빠진다면 스레드를 다시 깨울 수 없을지도 모른다.
* 대기 후에 조건을 검사하여 조건이 만족되지 않았다면 다시 대기하여 안전 실패를 막는다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 Lock이 보호하는 불변식이 깨질 위험이 있다.
* 위와같이 처리하지 않으면 조건이 만족되지 않아도 스레드가 깨어나는 상황이 발생할 수 있다.
* 스레드가 notify를 호출하여 대기 중인 스레드가 깨어나는 사이 다른 스레드가 Lock을 얻어 상태를 변경하는 경우
* 조건이 만족되지 않았지만 다른 스레드가 실수 혹은 악의적으로 notift를 호출하는 경우(외부에 공개된 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.)
* 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출하는 경우(notifyAll은 모든 스레드를 깨운다.)
* 대기 중인 스레드가 허위 각성(spurious wakeup) 한 경우 - notify 없이 깨어나는 경우

### notify와 notifyAll

* `notify`: 하나의 스레드를 깨운다.
* `notifyAll`: 모든 스레드를 깨운다. 가급적 notifyAll 사용을 권장한다.
  * notifyAll을 사용하면 깨어나야 할 모든 스레드가 깨어남을 보장하여 항상 정확한 결과를 얻을 수 있으며 안전하고 합리적이다.
  * notifyAll로 인해 다른 스레드까지 깨어날 수 있지만, 프로그램의 정확성에 영향을 주진 않는다. 깨어난 스레드들은 조건 확인 후, 충족되지 않았다면 다시 대기 상태로 변경될 것이다. (위와 같이 wait과 loop을 같이 사용해 조건 확인을 하는 경우!)
  * 모든 스레드가 같은 조건을 기다리고 있고, 조건 충족 시 하나의 스레드만 혜택을 받을 수 있다면 notify로 최적화할 수 있다.
  * notifyAll을 사용해 **관련 없는 스레드가 wait를 호출하는 공격으로부터 보호**할 수 있다. 만약 반드시 깨어나야 할 스레드가 notify를 삼켜버린다면 영원히 대기할 수 있기 때문이다.
