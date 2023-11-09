# item 78) 공유 중인 가변 데이터는 동기화해 사용하라

## synchronized 키워드

```java
public synchronized void createNewTab(final String webSiteName) {
    ...
}
```

* **해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.**
* 한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락을 건다.
* 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정하여 일관된 상태에서 다른 일관된 상태로 변화시킨다.
* 동기화를 제대로 사용하면 항상 일관된 상태를 볼 수 있다.
* **동기화 없이는 어떤 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.**
* 동기화된 메서드나 블록에 들어간 스레드는 같은 락의 보호하에 수행된 최종 결과를 보여준다.

## 변수의 읽기/쓰기는 원자적

* long, double 외에 변수를 읽고 쓰는 동작은 원자적이다.
* 따라서 여러 스레드가 같은 변수를 동기화없이 수정해도 정상적으로 저장한 값을 읽어올 수 있다.
* 스레드가 필드를 읽을 때 항상 수정이 반영된 값을 얻음을 보장하지만, **한 스레드가 저장한 값이 다른 스레드에 보이는지는 보장하지 않는다.**

따라서 **배타적 실행과 스레드 간의 안정적인 통신을 위해 동기화가 반드시 필요**하다.

## 동기화는 읽기/쓰기 모두 적용할 것

* 스레드1은 자신의 boolean 필드가 true가 되면 멈추도록 하고, false로 초기화해둔다.
* 스레드2는 boolean 필드를 false로 초기화해두고, 스레드 1을 멈추고자 할 때 true로 변경한다.
* 동기화하지 않으면 스레드2가 수정한 값이 스레드1의 값에 언제 반영될 지 모른다.
* 이 때 boolean 필드값을 수정(read/write)하는 메서드가 동기화되어있어야 한다.

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

* 쓰기와 읽기 모두 동기화되지 않으면 정상 동작을 보장하지 않는다.

## volatile 한정자

* 배타적 수행과는 상관 없으나, 항상 최근에 기록된 값을 읽음을 보장한다.
* 자바 스레드는 필요한 데이터를 빠르게 접근하기 위해 메인 메모리에서 읽은 데이터를 CPU 캐시에 두고 사용한다.
* volatile 한정자를 사용할 경우 CPU 캐시가 아닌 메인 메모리에 접근해 데이터를 사용하게 된다.
* 따라서 스레드 간의 안정적인 통신을 지원한다.
* 하지만 동기화 없이는 올바르게 동작하지 않는 경우도 있다.
* 아래 예제는 synchronized 한정자가 붙지 않은 메서드가 여러 스레드에서 불릴 경우, 같은 값이 여러 스레드에서 사용되어 중복된 nextSerialNumber 값이 사용될 수 있는 코드이다.
* nextSerialNumber를 `++` 할 때 필드에 접근하여 값을 읽은 후 증가한 값을 저장하는데, 이 작업 사이에 다른 스레드에서 필드에 접근하면 SerialNumber가 중복될 수 있다.

```cpp
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

* 따라서 메서드에 synchronized를 붙이고, 필드의 volatile을 제거해주어야 한다.

```java
private static int nextSerialNumber = 0;

public static synchronized int generateSerialNumber() {
    return nextSerialNumber++;
}
```

## atomic 패키지

* AtomicLong 등을 담은 `java.util.concurrent.atomic` 패키지는 락 없이도 thread-safe한 프로그래밍을 지원하는 클래스들을 제공한다.
* 성능 역시 동기화 버전보다 우수하다.
* 스레드 간 안정적인 통신과 배타적 실행을 지원한다.

```cpp
private static final AtomicLong nextSerialNumber = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNumber.getAndIncrement();
}
```

## 정리

* 애초에 가변 데이터를 스레드끼리 공유하지 않는 것이 좋다.
* 가변데이터는 단일 스레드에서만 쓰고, 불변 데이터만 공유하거나 아무것도 공유하지 말자.
* 안전 발행을 통해 객체를 다른 스레드에 안전하게 건넬 수 있다.
