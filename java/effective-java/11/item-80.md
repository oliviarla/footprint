# item 80) 스레드보다는 실행자, 태스크, 스트림을 애용하라

## 실행자 서비스

* java.util.concurrent 패키지에는 동시성 제어와 관련된 클래스들을 담고 있다.
* 특히 **실행자 프레임워크**라고 하는 인터페이스 기반 유연한 태스크 실행 기능을 담고 있다.
* 아래와 같이 간단하게 작업 큐를 생성하고 실행할 수 있다.

```java
// 작업 큐를 생성한다.
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행할 태스크를 넘긴다.
exec.execute(runnable);

// 실행자를 종료한다.
exec.shutdown();
```

### 주요 기능

* get() 메서드를 통해 태스크가 완료되기를 기다린다.

```java
ExecutorService exec = Executors.newSingleThreadExecutor();

exec.submit(() -> s.removeObserver(this)).get();
```

* 태스크들 중 하나라도 혹은 모든 태스크가 완료되기를 기다린다.

```java
// 태스크 모음 중 어느 하나를 기다린다.
exec.invokeAny(tasks);

// 모든 태스크가 완료되기를 기다린다.
exec.invokeAll(tasks);
```

* 실행자 서비스가 종료되기를 기다린다.

```java
exec.awaitTermination(10, TimeUnit.SECONDS);
```

* 완료된 태스크의 결과를 차례로 받는다.

```java
ExecutorService exex = Executors.newFixedThreadPool(2);
ExecutorCompletionService executorCompletionService =
    new ExecutorCompletionService(exex);

for (int i = 0; i < 2; i++) {
    executorCompletionService.take().get();
}
```

* 태스크를 특정 시간 혹은 주기적으로 실행하게 한다.

```java
ScheduledThreadPoolExecutor scheduledExecutor =
                new ScheduledThreadPoolExecutor(1);
```

* 큐를 둘 이상의 스레드가 처리하게 하고 싶다면 다른 정적 팩터리를 이용해 다른 종류의 실행자 서비스를 생성하면 된다. 스레드의 개수는 고정할 수 있고(newFixedThreadPool), 필요에 따라 늘어나거나 줄어들게도(newCachedThreadPool) 설정할 수 있다.

### 실행자 서비스 생성 방법

* `java.util.concurrent.Executors`의 정적 팩터리들을 이용하거나, ThreadPoolExecutor 클래스를 직접 사용해도 된다.

#### ThreadPoolExecutor

* 자바에서 ThreadPool 사용을 위해 지원하는 클래스

> ThreadPool: 쓰레드를 미리 만들어 놓고 재사용하는 방식

* 평소에는 **corePoolSize**만큼 기본 스레드 개수를 유지하다가, 모든 코어 쓰레드가 바쁘고 내부 큐도 꽉차면 **maximumPooSize** 까지 개수를 늘린다.
* 더 알아보려면 링크를 참고하는 것도 좋다. [https://keichee.tistory.com/388](https://keichee.tistory.com/388)

#### newCachedThreadPool vs newFixedThreadPool

* 두 실행자 서비스 생성 방법은 Executors의 정적 팩터리에서 제공되는 방식이다.
* **newCachedThreadPool**
  * 작은 프로그램이나 가벼운 서버에 적합
  * 요청받은 태스크들을 큐에 쌓지 않고 즉시 스레드에 위임해 실행
  * 사용 가능한 스레드가 없다면 새로 스레드를 생성하기 때문에 CPU 사용률이 100%에 치닫는 상황이 생길 수 있다. 새로운 태스크가 도착할 때마다 가용 스레드가 없으니 다른 스레드를 계속 생성하면 상황은 더욱 악화될 것이다.
* &#x20;**newFixedThreadPool**
  * 무거운 프로덕션 서버에 적합
  * 스레드 개수를 고정해두고 사용

## 정리

* 작업 큐를 손수 만들거나 스레드를 직접 다루는 일은 삼가야 한다.
* 스레드를 직접 다루면 스레드가 작업 단위와 수행 매커니즘 역할을 모두 수행하지만, 실행자 프레임워크를 사용하면 작업 단위와 실행 매커니즘이 분리된다.
* 실행자 서비스에 태스크 수행을 맡기면, **실행자 프레임워크가 원하는 태스크 수행 정책에 따라 작업 수행을 담당**해주고, **실제 작업 수행은 각 스레드에 위임**되는 것이다.
* 실행자 프레임워크인 ForkJoinPool을 구성하는 스레드들은 태스크들(ForkJoinTask)을 처리하며, 수행이 끝난 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수 있다. 이를 통해 모든 스레드를 바쁘게 움직여 CPU를 최대한 활용해 높은 처리량과 낮은 지연시간을 가질 수 있다.
