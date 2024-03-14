# Thread

## 생성 방법

* Thread 클래스를 상속받거나 Runnable 인터페이스를 구현하여 Thread를 생성할 수 있다.
* 다음은 thread 클래스를 상속받아 새로운 Thread를 생성하는 방법이다.

```java
class PrimeThread extends Thread {
  long minPrime;
  PrimeThread(long minPrime) {
    this.minPrime = minPrime;
  }

  public void run() {
    // compute primes larger than minPrime
     . . .
  }
}
```

* 다음은 Runnable 인터페이스를 구현하여 Thread를 생성하는 방법이다.

```java
class PrimeRun implements Runnable {
  long minPrime;
  PrimeRun(long minPrime) {
    this.minPrime = minPrime;
  }

  public void run() {
    // compute primes larger than minPrime
     . . .
  }
}
```

## 실행 방법

* start() 메서드를 통해 새로운 스레드에서 작업을 수행할 수 있다.

```java
PrimeThread primeThread = new PrimeThread(13);
primeThread.start();
```

* start() 메서드를 호출하면 새로운 스레드에서 스택 프레임을 생성하고 run 메서드를 호출한다. 작업이 완료되면 값을 반환하고 스택 프레임을 콜 스택에서 제거한다.
* run() 메서드도 호출할 수 있지만 현재 스레드에서 작업을 수행한다. 따라서, 스레드 클래스를 상속해 새로운 스레드에서 작업을 수행하려는 의미와 부합하지 않는다.

## WAITING 상태 만들기

* Thread를 WAITING 상태로 만드는 방법은 아래와 같이 3가지가 존재한다.

1.  **Object.wait()**

    스레드가 객체의 모니터를 소유할 때 , 현재 스레드를 `wait()` 명령어로 멈추고 다른 스레드의 작업이 완료되면 `notify()` 메서드를 호출해 다시 이 스레드가 깨어나도록 할 수 있다.
2.  **Thread.join()**

    워커 스레드 객체의 `join()` 메서드를 호출하여 현재 스레드에서 실행시킨 워커 스레드가 종료되기를 기다릴 수 있다.
3.  **LockSupport.park()**

    `LockSupport.park()` 메서드를 사용해 현재 스레드를 WAITING (parking) 상태로 만들 수 있다. 단, low-level의 API이고 잘못 사용되어 데드락에 빠질 위험이 존재하므로 일반적인 상황에서는 Thread 클래스의 wait / join 메서드를 사용하는 것이 좋다. 이 방식을 사용하려면 직접 블록된 스레드와 대기중인 스레드를 관리해주어야 한다.

