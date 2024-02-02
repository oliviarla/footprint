# Interrupt

### Interrupt란

* Java에서는 실행중인 특정 스레드를 종료하기 위해 대상 스레드 객체의 interrupt() 메서드를 호출하여 Interrupt를 걸 수 있다.
* 단, 자바에서 제공해주는 특정 클래스의 메서드를 사용하는 것이 아닌 직접 스레드에서 태스크를 수행할 때 Interrupt여부에 따라 종료시키도록 하지 않으면, thread.interrupt() 메서드를 호출해도 스레드가 종료되지 않을 수 있다.
* 예를 들어 아래와 같이 cancel 메서드를 호출해 무한 while루프를 빠져나오고자 하더라도 스레드 내부 동작에서 interrupt 여부를 확인하여 종료시키는 로직이 존재하지 않는다면, 작업이 종료될 수 없다.

```java
Executor executor = Executors.newSingleThreadExecutor();
Future<?> future = executorService.submit((Runnable) () -> {
  while (true) {
    System.out.println("hello");
  }
});
future.cancel(true);
```

* 따라서 아래와 같이 interrupt되었을 때에는 작업을 종료하도록 코드를 작성해야 interrupt되었을 떄 스레드를 종료하게 된다.

```java
Executor executor = Executors.newSingleThreadExecutor();
Future<?> future = executorService.submit((Runnable) () -> {
  while (!Thread.currentThread().isInterrupted()) {
    System.out.println("hello");
  }
});
future.cancel(true);
```

* 다음 메서드들은 외부에 의해 interrupt되면 스레드의 Interrupt 상태를 초기화시키고, 스레드를 종료시키고, `InterruptedException` 을 발생시킨다.
  * &#x20;`Object` 클래스의 `wait()`, `wait(long)`, `wait(long, int)`
  * `Thread` 클래스의 `join()`, `join(long)`, `join(long, int)`, `sleep(long)`, `sleep(long, int)`

### InterruptedException 처리 방법

* `InterruptedException`은 Checked Exception이기 때문에 반드시 try-catch문으로 처리해주거나 메서드 시그니처에 명시해 메서드 호출자가 처리하도록 해야 한다.
* 만약 아래와 같이 스레드 태스크(Runnable) 내부에서 `Thread.sleep(n)`을 호출했고, 이 때 `InterruptedException`이 발생한 경우, 예외가 발생하면서 interrupt 상태가 초기화되었으므로 interrupt 상태를 현재 스레드에 전파시켜 Runnable 작업이 끝나도록 할 수 있다.

```java
Executor executor = Executors.newSingleThreadExecutor();
Future<?> future = executorService.submit((Runnable) () -> {
    while (!Thread.currentThread().isInterrupted()) {
        try {
            System.out.println("interruptible");
            Thread.sleep(1000);
        } catch (InterruptedException ex) {
            Thread.currentThread().interrupt();
        }
    }
});
```

#### 출처

[https://easywritten.com/post/best-way-to-shutdown-executor-service-in-java/](https://easywritten.com/post/best-way-to-shutdown-executor-service-in-java/)
