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
