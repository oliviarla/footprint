# 15장: CompletableFuture와 Reactive 개요

* MSA가 도입된 후 작은 서비스 간의 네트워크 통신이 증가하였다. 특정 서비스나 데이터베이스에 요청을 보내고 응답을 기다리면서 스레드가 블록되는 것은 자원의 낭비이다.
* 애플리케이션의 생산성을 극대화하려면 코어를 바쁘게 유지해야 하므로 **동시성**을 확보하는 것이 중요하다.
  * 예를들어 한 코어에서 작업1을 수행하며 원격 서비스에 요청을 보내두고 응답을 기다리는 동안, 가만히 있지 않고 작업2를 수행한다.
  * 작업1에서 보냈던 요청에 대한 응답이 오면 작업2를 끝내고 작업1을 마무리한다.

> 병렬성은 여러 코어에서 나누어 처리하는 것이고, 동시성은 한 코어에서 여러 작업을 CPU가 쉬지않고 번갈아가며 수행하는 것이다.

<figure><img src="../../.gitbook/assets/image (164).png" alt=""><figcaption><p><a href="https://seamless.tistory.com/42">https://seamless.tistory.com/42</a></p></figcaption></figure>

## 🏝️ 자바 동시성의 진화 과정

### 버전 별 변화

* Java 초창기: Runnable, Thread, Synchronized Classes
* Java 5: ExecutorService, Callable\<T>, Future\<T>, Generic
  * ExecutorService 인터페이스는 Executor 인터페이스를 상속받으며, Callble을 실행하는&#x20;
* Java 7: RecursiveTask (ForkJoin을 지원)
* Java 8: CompletableFuture
* Java 9: 분산 비동기 프로그래밍(Reactive), Flow 인터페이스

### 스레드의 높은 추상화

* 아래와 같이 작성할 경우 한 개의 코어에서 하나의 스레드만으로 동작하게 된다.

```java
long sum = 0;
for (int i=0;i<1_000_000; i++) {
    sum += stats[i];
}
```

* 아래와 같이 작성할 경우, stats 배열의 모든 요소를 더할 때 여러 스레드에서 병렬로 처리하는 것을 단순하게 추상화할 수 있다.
* 자바 코드로 직접 스레드를 할당해서 sum하는 과정(외부 반복) 대신 내부 반복을 통해 쉽게 병렬성을 달성한다.

```java
sum = Arrays.stream(stats).parallel().sum();
```

### Executor와 Thread Pool

* 프로그래머가 태스크 제출과 실행을 분리할 수 있는 기능을 제공한다.

#### 스레드의 문제

* 자바 스레드는 직접 운영체제 스레드에 접근하는데, 이 스레드는 만들고 종료하는 cost가 비싸다. 게다가 운영체제 스레드 개수는 제한되어 있기 때문에, 기존 스레드가 실행되는 상태에서 계속 새로운 스레드 만드는 상황이 발생하면 문제가 생길 수 있다.
* 자바의 스레드 개수가 하드웨어 스레드 개수보다 많기 때문에 일부 운영체제 스레드가 블록되거나 쉬는 상황에서 모든 하드웨어 스레드가 코드를 실행하도록 할당되는 상황이 발생 가능하다.
  * ex) 인텔 코어&#x20;
* 프로그램에서 사용할 최적의 자바 스레드 개수는 가용 하드웨어 코어 개수에 따라 결정된다.

#### 스레드 풀

* Java에서는 ExecutorService 인터페이스를 제공해 스레드 풀을 활용할 수 있도록 한다.
* 프로그래머는 ExecutorService에 태스크(Runnable/Callable)를 제출하면 스레드에서 수행된 결과를 나중에 수집할 수 있다.
* ExecutorService.newFixedThreadPool 을 사용하면 워커스레드라고 불리는 nThreads를 포함하는 ExecutorService를 만들어 스레드 풀에 저장한다.
* 스레드 풀에서는 사용 가능한 스레드에 태스크를 먼저온 순서대로 할당해 실행한다. 태스크가 종료되면 스레드를 스레드 풀에 반납한다.
* 스레드 풀 사용 시 하드웨어에 맞는 수의 태스크를 유지함과 동시에 수 천개의 태스크를 스레드 풀에 아무 오버헤드 없이 제출할 수 있고, 큐의 크기 조정, 거부 정책, 태스크 종류에 따른 우선순위 등 다양한 설정을 할 수 있다.

#### 스레드 풀의 주의점

* k개의 스레드를 가진 스레드 풀은 k개까지의 스레드만 동시에 실행 가능하다. 따라서 k개를 초과하는 숫자의 태스크가 들어오면 큐에 저장하고 다른 태스크가 종료되어야만 스레드에 할당된다.
* 이러한 특성으로 인해 I/O나 네트워크 연결을 기다리거나 Sleep하는 등 **block될 수 있는 태스크는 스레드 풀에 제출하지 않는 것이 좋다**.
* I/O를 기다리거나 네트워크 연결을 기다리는 태스크가 있다면 스레드를 차지하게 되어 작업 효율성이 낮아질 수 있다.
* 혹은 처음 제출한 태스크가 나중의 태스크 제출을 기다리는 상황에 빠지면 데드락에 걸릴 수도 있다.
* 프로그램을 종료하기 전 스레드풀을 종료해야 한다. 스레드 풀의 워커 스레드가 만들어진 다음 다른 태스크 제출을 기다리면서 종료되지 않는 상황이 발생할 수 있다.
* 자바는 오랫동안 실행되는 ExecutorService를 갖는 경우를 위해 `Thread.setDaemon()` 메서드를 제공한다.

#### 중첩되지 않은 메서드 호출

* 7장에서 다룬 fork/join 방식은 아래와 같이 중첩된 메서드 호출 방식이다. 이번 15장에서는 중첩되지 않은 메서드에 대해 다룰 예정이다.
* 7장에서의 fork/join 방식에는 엄격한 포크/조인 방식과 여유로운 포크/조인 방식이 존재한다.
* 엄격한 포크/조인: 태스크나 스레드가 메서드 호출 안에서 시작되면, 해당 메서드는 모든 작업이 끝난 후에야 결과를 반환한다. 스레드 생성과 `join()` 이 한 쌍처럼 중첩된 메서드 호출 내에 추가된다.

<figure><img src="../../.gitbook/assets/image (165).png" alt=""><figcaption><p>엄격한 포크/조인</p></figcaption></figure>

* 여유로운 포크/조인: 시작된 태스크를 외부 호출에서 종료하도록 기다리는 방식도 비교적 안전하다. 사용자는 이 인터페이스를 일반 호출로 간주할 것이다.

<figure><img src="../../.gitbook/assets/image (166).png" alt=""><figcaption><p>여유로운 포크/조인</p></figcaption></figure>

* 15장에서는 사용자의 메서드 호출에 의해 스레드가 생성되고, 메서드를 벗어나서 계속 스레드에서 작업이 수행되는 동시성 형태에 초점을 맞춘다.
* 즉 메서드가 반환된 후에도 별도 스레드에서 태스크를 계속 실행하는 메서드인 **비동기 메서드**를 다루게 된다.

<figure><img src="../../.gitbook/assets/image (167).png" alt=""><figcaption><p>비동기 메서드</p></figcaption></figure>

* 비동기 메서드의 위험성
  * 스레드 실행은 메서드를 호출한 다음의 코드와 동시에 실행되므로 데이터 경쟁 문제를 일으키지 않도록 주의해야 한다.
  * 기존 실행 중이던 스레드가 종료되지 않은 상황에서 자바의 main() 메서드가 종료되면, 두 가지 방식으로 종료된다. 하지만 이 두 방식은 모두 안전하지 못하다.
    * 애플리케이션 종료하지 못하고 모든 스레드가 실행을 끝낼때까지 기다린다.
      * 종료되지 않은 스레드에 의해 애플리케이션 충돌이 발생할 수 있다.
    * 애플리케이션 종료를 방해하는 스레드를 강제 종료시키고 애플리케이션을 종료시킨다.
  * 앞서 말했듯, 스레드 풀을 포함한 모든 스레드를 종료시킨 후 애플리케이션을 종료하는 것이 좋다.
* 데몬 스레드
  * 자바의 스레드는 데몬/비데몬 스레드로 나뉜다.
  * 데몬 스레드는 애플리케이션 종료 시 같이 강제종료되어 디스크의 데이터 일관성을 파괴하지 않는 동작을 수행할 때 유용하다.
  * 비데몬 스레드의 경우 종료될 때 까지 애플리케이션이 종료되지 못한다.

#### 스레드를 통한 목표

* 모든 하드웨어 스레드를 활용해 병렬성의 장점을 극대화하여 프로그램을 효율적으로 동작시키는 것이 목표이다.
* 이를 위해서는 프로그램을 적당히 작은 태스크 단위로 구조화해야 한다. (너무 작은 크기로 나누면 태스크 변환 비용이 발생하기 때문에 좋지 않다.)

## 🏝️ 동기 API, 비동기 API

* 병렬성의 유용함
  * 외부반복(for 루프)을 내부반복(stream 메서드)로 변경하여 자바 런타임 라이브러리가 복잡한 스레드 작업을 하지 않고 병렬로 요소가 처리되도록 할 수 있다.
  * 개발자가 직접 for 루프가 실행되는 시점을 추측해서 프로그래밍하는 것보다, 런타임 시스템에서 정확히 사용할 수 있는 스레드를 확인하고 수행해주는 것이 좋다.

### 기본 스레드 제어 방식

* 아래와 같이 f, g 두 메서드의 호출을 합해야 하는데 f,g 메서드의 수행 시간이 오래걸린다고 가정한다.

```java
int f(int x);
int g(int x);

int y = f(x);
int z = g(x);
System.out.println(y + z);
```

* f와 g를 별도의 CPU 코어로 실행하면 둘 중에서 오래걸리는 작업의 시간만을 사용해 합계를 구할 수 있으므로, f와 g를 순차적으로 수행해 합계를 구하는 것보다 훨씬 시간이 적게 든다.

```java
int x = 1337;
Result result = new Result();

Thread t1 = new Thread(() -> { result.left = f(x); });
Thread t2 = new Thread(() -> { result.right = g(x); });
t1.start();
t2.start();
t1.join();
t2.join();
System.out.println(result.left + result.right);
```

### Future 사용 방식

* Runnable을 사용하는 대신 Future API 인터페이스를 이용해 코드를 단순화할 수 있다.

```java
int x = 1337;

ExecutorService executorService = Executors.newFixedThreadPool(2);
Future<Integer> y = executorService.submit(() -> f(x));
Future<Integer> z = executorService.submit(() -> g(x));
System.out.println(y.get() + z.get());

executorService.shutdown();
```

* 혹은 아예 f, g 메서드의 시그니처를 변경할 수 있다. 이를 통해 작고 합리적인 크기의 여러 태스크로 나누어 병렬 하드웨어로 프로그램 실행 속도를 극대화할 수 있다.

```java
Future<Integer> f(int x);
Future<Integer> g(int x);

Future<Integer> y = f(x);
Future<Integer> z = g(x);
System.out.println(y.get() + z.get());
```

### 리액티브 방식

* f, g의 시그니처를 바꿔 콜백 형식의 프로그래밍을 이용할 수도 있다.
* f에 콜백(람다)을 전달해 내부적으로 메서드 수행 결과가 반환되면 콜백을 호출하여 결과를 출력하도록 할 수 있다.
* 하지만 f,g 중 먼저 수행된 것과 나중에 수행된 것에서 2번 출력되므로 기존 코드의 수행 결과와 달라진다.

```
void f(int x, IntConsumer dealWithResult);
```

```java
int x = 1337;
Result result = new Result();

f(x, (int y) -> {
  result.left = y;
  System.out.println(result.left + result.right);
});


g(x, (int z) -> {
  result.right = z;
  System.out.println(result.left + result.right);
});
```

* 리액티브 형식의 비동기 API는 보통 한 결과가 아니라 일련의 이벤트에 반응하도록 설계되어 나중에 스트림으로 연결하기에 적절하고 Future형식의 API는 일회성의 값을 처리하는 데에 적절하다.

### 블로킹

* sleep()
  * 스레드는 sleep하여도 시스템 자원을 점유하고 있으므로, 스레드가 점점 많아지는데 sleep상태인 스레드의 비율이 늘어난다면 심각한 문제를 초래한다.
  * sleep은 다른 태스크가 수행되지 못하도록 막고, 이 sleep 상태의 태스크는 외부에서 중지시키지 못한다.
* 블로킹 동작에는 다른 태스크가 어떤 동작을 완료하기를 기다리는 동작(`Future.get()`)  외부 상호 작용(DB, 키보드 입력 등)이 해당된다.
* ScheduledExecutorService
  * 특정 태스크를 수행하는 작업이 오래걸릴 것으로 예상될 때 해당 작업이 끝날 시점에 새 태스크를 실행하도록 할 수 있다.
  * 아래는 work1()을 수행한 다음 작업을 종료하고, work2()가 10초 후에 수행될 수 있도록 작업을 할당해둔다.

```java
public class ScheduledExecutorServiceExample {
  public static void main(String[] args) {
    ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);
    
    work1();
    scheduledExecutorService.schedule(ScheduledExecutorServiceExample::work2, 10, timeUnit.SECONDS);
    
    scheduledExecutorService.shutdown();
  }

  public static void work1() {
    ...
  }

  public static void work2() {
    ...
  }
}
```

* 사실상 아래 방식과 동작은 같지만, 아래 방식은 Thread.sleep()하는동안 스레드 자원을 점유하고 있지만,  위 방식은 다른 작업이 실행될 수 있도록 허용한다. 스레드에는 제한이 있고 비싼 연산이므로 블록해야 하는 태스크가 여러 개 있다면 위와 같은 방식을 사용하자.

```java
work1();
Thread.sleep(10000);
work2();
```

* 태스크가 실행되면 자원을 점유하므로, 태스크가 블록된 상태여도 자원이 해제되지 않고 계속 태스크가 실행된다.
* 태스크를 블록하는 것보다는 다음 작업을 태스크로 제출하고 현재 태스크는 종료하는 것이 좋다.
* I/O 작업의 경우에도 읽기 시작 메서드를 호출하고 읽기 작업 완료 시 처리할 태스크를 런타임 라이브러리에 스케줄하도록 요청하고 종료한다.
* CompletableFuture의 경우 get()으로 명시적으로 블록하는 것이 아닌 콤비네이터를 사용해 런타임 라이브러리 내에 추상화한다.

### 현실성 확인

* 시스템을 다수의 작은 태스크로 설계해 블록할 수 있는 모든 동작을 비동기 호출로 구현한다면 병렬 하드웨어를 최대한 활용 가능하다.
* 하지만 현실적으로 모든 것을 비동기화 할 수는 없다.
* 모든 API를 비동기로 만드는 것보다는 자바의 개선된 동시성 API를 이용해보는 것이 좋다.
* 네트워크 서버의 블록/논블록 API를 일관적으로 제공하는 Netty 라이브러리를 사용해보는 것도 좋다.

### 예외 처리

* Future, 리액티브 형식의 비동기 API에서 호출된 메서드의 내부는 별도 스레드에서 호출되어 이 때 발생한 에러는 호출자의 실행 범위를 벗어난다.
* CompletableFuture에서는 `get()` 메서드에 예외를 처리하는 기능을 제공하고, 예외에서 회복할 수 있도록 `exceptionally()` 메서드도 제공한다.
* 리액티브 형식의 비동기 API에는 정상적일 때의 콜백과 예외 상황일 때의 콜백을 함께 인자로 주어야 한다.

```java
void f(int x, Consumer<Integer> dealWithResult, Consumer<Throwable> dealWithException);
```

* Subscriber
  * 자바 9 Flow 인터페이스 내의 Subscriber는 아래와 같이 Error상황일 때 호출될 메서드를 Override할 수 있다.
  * 키보드 장치에서 숫자를 읽어오는 작업을 생각해보면, 숫자가 입력되면 onNext를 호출하고, 잘못된 형식이 입력되면 onError를 호출하고, 더이상 처리할 데이터가 없으면 onComplete를 호출하면 된다.
  * 이런 종류의 호출을 메시지 또는 이벤트 라고 부른다.

```java
public static interface Subscriber<T> {
        public void onSubscribe(Subscription subscription);        
        public void onNext(T item);
        public void onError(Throwable throwable);
        public void onComplete();
}
```

## 🏝️ 박스와 채널 모델

### 개념

* 동시성 모델을 잘 설계화하고 개념화하기 위한 모델
* 대규모 시스템 구현의 추상화 수준을 높일 수 있다.
* 병렬성을 직접 프로그래밍하는 관점으로부터 콤비네이터를 통해 내부적으로 작업을 처리하는 관점으로 바꿔준다.

### 예시

* x라는 변수를 p 함수에 넣고, 그 결과를 q1, q2 함수에 적용해 반환된 값을 r 함수에 넣어 최종 결과를 얻는 태스크는 아래와 같이 표현할 수 있다.

<figure><img src="../../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

*   위 예시를 바로 자바로 구현하면 병렬성 활용과는 거리가 먼 코드가 된다.

    ```java
    int t = p(x);
    System.out.println(r(q1(t), q2(t));
    ```
*   Future를 이용하려면, `p 함수`는 가장 먼저 처리되어야 하고, `r 함수`는 가장 나중에 처리되어야 하므로 Future 활용이 불가능하고, `q1`, `q2 함수`만 Future 활용이 가능하다.

    ```java
    int t = p(x);
    Future<integer> a1 = executorService.submit(() -> q1(t));
    Future<integer> a2 = executorService.submit(() -> q2(t));
    System.out.println(r(a1.get(), a2.get());
    ```
* 시스템이 커지고  많은 박스와 채널 다이어그램이 등장하고 각 박스가 내부적으로 자신만의 박스와 채널을 사용하게 되면, 많은 태스크에서 get() 메서드로 Future에서 결과 반환을 대기하게 될 수 있다.
*   CompletableFuture와 콤비네이터를 이용해 다른 Function들을 조합할 수 있다.

    ```java
    p.thenBoth(q1, q2).thenCombine(r)
    ```

## 🏝️ CompletableFuture와 콤비네이터를 이용한 동시성

* 여러 Future를 조합할 수 있는 기능이 추가된 Future 인터페이스의 구현체
* ComposableFuture라고 부르지 않는 이유는 실행할 코드 없이도 Future를 만들 수 있고, complete() 메서드를 통해 나중에 어떤 값을 이용해 다른 스레드가 완료할 수 있고 get()으로 값을 얻을 수 있기 때문이다.
* f(x)와 g(x)를 동시에 실행해 합계를 구하는 코드를 아래 두 방식으로 작성할 수 있지만, f(x)나 g(x)의 실행이 끝나지 않으면 get()을 기다려야 한다.

```java
ExecutorService executorService = Executors.newFiexedthreadPool(10);
int x = 1337;

// f(x)는 별도 스레드에서, g(x)는 메인 스레드에서 수행
CompletableFuture<Integer> a = new CompletableFuture<>();
executorService.submit(()-> a.complete(f(x)));
int b = g(x);
System.out.println(a.get()+b);

executorService.shutdown();
```

### thenCombine

* 다른 CompletableFuture와 두 CompletableFuture의 결과를 어떻게 변환할지에 대해 BiFunction 형태로 입력받는다.
* 아래와 같이 a CompletableFuture의 thenCombine 메서드에 b CompletableFuture와 BiFunction 형태의 람다를 입력받아 새로운 c CompletableFuture를 생성한다.
* c CompletableFuture는 a, b의 작업이 완료되어야만 스레드에서 실행이 시작된다. 따라서 이전 코드와 달리 블로킹이 발생하지 않는다.
* 이전 코드에서는 f(x), g(x)의 결과를 둘 중 하나를 실행시킨 스레드에서 합한 반면, 아래 코드는 f(x), g(x), 결과 합치는 연산을 모두 다른 스레드에서 수행하게 된다.

```java
ExecutorService executorService = Executors.newFiexedthreadPool(10);
int x = 1337;

CompletableFuture<Integer> a = new CompletableFuture<>();
CompletableFuture<Integer> b = new CompletableFuture<>();

CompletableFuture<Integer> c = a.thenCombine(b, (y, z) -> y + z);
executorService.submit(()-> a.complete(f(x)));
executorService.submit(()-> b.complete(g(x)));

System.out.println(c.get());
executorService.shutdown();
```

* 많은 수의 Future를 사용하는 경우 CompletableFuture와 콤비네이터를 이용해 get()에서 블록되지 않도록 하여 병렬 실행의 효율성을 높이고 데드락을 피할 수 있다.

## 🏝️  발행-구독 그리고 리액티브 프로그래밍

* Future는 독립적 실행과 병렬성에 기반하므로, 한 번만 실행해 연산이 끝나면 결과를 제공한다.
* 리액티브 프로그래밍은 시간이 흐르면서 여러 Future 같은 객체를 통해 여러 결과를 제공하고, 가장 최근의 결과에 대해 반응(react)하는 부분이 존재한다.
* 계산기 객체 같이 매 초마다 온도 값을 제공하거나 웹 서버 컴포넌트 응답을 기다리는 리스너 객체를 사용하는 경우 계속해서 결과를 반환하기 때문에 여러 번의 결과를 처리해줄 방법이 필요하다.
* 이 결과 중 가장 최근의 결과가 중요한 경우가 많다.
* 자바9에서는 java.util.concurrent.Flow 인터페이스에 발행-구독 모델을 적용해 리액티브 프로그래밍을 제공한다.
  * **구독자**(Subscriber)가 구독할 수 있는 **발행자**(Publisher)
  * 이 연결을 **구독**(subscription)이라 한다.
  * 이 연결을 이용해 **메시지**(또는 **이벤트**)를 전송한다.

### 간단한 예제

* 예제에서는 간단한 Publisher와 Subscriber 인터페이스를 작성하고, 이를 구현하는 SimpleCell 클래스를 만들어본다.
* Publisher는 구독자를 등록할 수 있는 메서드를 제공한다.
* Subscriber는 publisher가 이벤트를 구독자에게 전달할 때 사용될 수 있도록 `onNext` 메서드를 제공한다.

```java
interface Publisher<T> {
  void subscribe(Subscriber<? super T> subscriber);
}

interface Subscriber<T> {
  void onNext(T t);
}
```

* SimpleCell 클래스는 다른 SimpleCell을 구독해 이벤트에 반응하는 Subscriber가 될 수도 있고, 그 자체로 Publisher가 될 수도  있다.

```java
public class SimpleCell implements Publisher<Integer>, Subscriber<Integer> {
  private int value = 0;
  private String name;
  private List<Subscriber> subscribers = new ArrayList<>();

  public SimpleCell(String name) {
      this.name = name;
  }

  @Override
  public void subscribe(Subscriber<? super Integer> subscriber) {
    subscribers.add(subscriber);
  }

  // 본 SimpleCell 객체가 구독한 다른 SimpleCell로부터 이벤트가 오면, 해당 이벤트 값을 value에 저장
  @Override
  public void onNext(Integer newValue) {
    this.value = newValue;
    System.out.println(this.name + ":" + this.value);
    // value 값이 변경되었으므로, 모든 구독자에게 value 값을 전달
    notifyAllSubscribers();
  }
  
  private void notifyAllSubscribers() {
    subscribers.forEach(subscriber -> subscriber.onNext(this.value));
  }
}
```

* 아래와 같이 실행하면, c3가 c1을 구독하므로, c3는 c1과 같은 value 값을 갖게 된다. 즉, c1, c3은 value가 10이 되고, c2는 value가 20이 된다.

```java
SimpleCell c3 = new SimpleCell("c3");
SimpleCell c2 = new SimpleCell("c2");
SimpleCell c1 = new SimpleCell("c1");

c1.subscribe(c3);
c1.onNext(10);
c2.onNext(20);
```

* 이번에는 SimpleCell을 확장하여 왼쪽 값과 오른쪽 값을 지니는 ArithmeticCell 클래스를 만든다.

```java
public class ArithmeticCell extends SimpleCell {
  private int left;
  private int right; 
    
  public ArithmeticCell(String name) {
    super(name);
  }
    
  public void setLeft(int left) {
    this.left = left;
    onNext(left + this.right);
  }
    
  public void setRight(int right) {
    this.right = right;
    onNext(right + this.left);
  }
}
```

* 아래와 같이 c1의 값이 바뀌면 c3의 left를 갱신하도록 하고 c2의 값이 바뀌면 c3의 right를 갱신하도록 하면, 두 값을 각각 갱신할 수 있다.

```java
ArithmeticCell c3 = new ArithmeticCell("c3");
SimpleCell c2 = new SimpleCell("c2");
SimpleCell c1 = new SimpleCell("c1");

c1.subscribe(c3::setLeft);
c2.subscribe(c3::setRight);

c1.onNext(10); // c1의 값을 10으로 갱신
c2.onNext(20); // c2의 값을 20으로 갱신
c1.onNext(15); // c1의 값을 15으로 갱신

```

* 데이터가 Publisher에서 Subscriber로 흐르기 때문에 notifyAllSubscribers() 를 통해 onNext()를 호출하면 다운 스트림이라고 부르며, onNext()를 외부에서 직접 호출하여 Publisher의 데이터를 갱신할 때에는 업스트림이라고 부른다.
* 실제로 Flow를 사용하려면 onError나 onComplete와 같은 메서드를 통해 예외 발생, 데이터 흐름 종료 등을 알수 있어야 한다.

### 역압력

* **압력**: 수많은 메시지(이벤트)가 매 초마다 `Subscriber`의 onNext로 전달되는 상황이 발생하면 처리에 어려움을 겪을 수 있다.
* `Publisher`가 메시지를 무한의 속도로 방출하는 것보다는, 요청했을 때에만 다음 메시지를 보내는 형태의 **역압력 기법**이 필요하다.
* `Publisher`의 subscribe 메서드에 `Subscriber`를 넣어 호출하면, 구독자에 포함될 수 있다.

```java
@FunctionalInterface
public static interface Publisher<T> {
    public void subscribe(Subscriber<? super T> subscriber);
}
```

* `Publisher`는 subscribe 메서드가 호출되면 `Subsciption` 객체를 만들어 `Subscriber`에게 전달한다.
* `Subscriber`는 `Subsciption` 객체를 내부 필드에 저장해두어, `Publisher`가 구독 정보를 참고하여 메시지를 보내도록 한다.

```java
public static interface Subscriber<T> {
    void onSubscribe(Subscription subscription);
    // ...
}

public static interface Subscription {
  public void cancel();
  public void request(long n);
}
```

* reactive pull-based 방식을 통해 `Subscriber`가 `Publisher`로부터 데이터를 요청해 받아오도록 하는 방식으로 구현할 수도 있다.

## 🏝️  리액티브 시스템 vs 리액티브 프로그래밍

* 리액티브 시스템은 런타임 환경이 변화에 대응하도록 전체 아키텍처가 설계된 프로그램을 가리킨다.&#x20;
* 반응성(responsive), 회복성(resilient), 탄력성(elastic)의 세가지 속성을 필요로 한다.
  * 반응성: 큰 작업을 처리하느라 간단한 질의 응답을 지연하지 않도록 하여 실시간으로 입력에 반응하는 것
  * 회복성: 한 컴포넌트의 실패가 전체 시스템에 영향을 주지 않는 것
  * 탄력성: 시스템이 자신의 작업 부하에 맞게 큐나 스레드를 조정하여 효율적으로 작업을 처리하도록 하는 것
  * 메시지 주도 속성: 박스와 채널 모델에 기반하여 처리할 입력을 기다리고 다른 컴포넌트로 보내면서 시스템이 반응하는 것&#x20;
* 리액티브 프로그래밍은 이러한 속성들을 구현하는 방법 중 하나이다.
