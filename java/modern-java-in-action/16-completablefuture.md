# 16장: CompletableFuture

## 🏝️ Future의 단순 활용

* Java 5 부터 미래의 어느 시점에 결과를 얻는 모델에 활용할수 있도록 Future 인터페이스를 제공하고 있다.
* 시간이 걸리는 작업들을 Future 내부에 두면, 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업들을 할 수 있다.
* 아래 코드는 Future로 다른 스레드에 오래걸리는 작업을 할당하고, 호출자 스레드에서는 future.get() 하기 전&#x20;

```java
ExecutorService executor = Executors.newCachedThreadPool();

Future<Double> future = executor.submit(new Callable<Double>() {
  public Double call() {
    return doSomeLongComputation();
  }
})

doSomeThingElse();

try {
  future.get(1, TimeUnit.SECONDS); // 비동기 작업의 결과가 준비되지 않았다면 1초까지 기다려본다.
} catch (ExecutionException ee) {
  // 계산 중 예외 발생
} catch (InterruptedException ie) {
  // 현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException te) {
  // Future가 완료되기 전 타임아웃 발생
}
```



<figure><img src="../../.gitbook/assets/image (51).png" alt="" width="563"><figcaption></figcaption></figure>

