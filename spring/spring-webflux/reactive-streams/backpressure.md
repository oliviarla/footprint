# Backpressure

### Backpressure 필요성

* Publisher로부터 받은 데이터를 Downstream Publisher, Subscriber 등에서 안정적으로 처리하기 위한 수단이다.
* Subscriber가 데이터를 처리하는 속도보다 Publisher가 데이터를 보내는 속도가 더 빠르다면, 데이터가 계속 쌓이게 되어 오버플로우가 발생하거나 최악의 경우 OOM 등의 이유로 인해 시스템이 다운될 수 있다.

### Backpressure 지원 방식

* 데이터 개수 제어
  * Subscriber가 적절히 처리할 수 있는 정도의 데이터 개수를 Publisher에 요청한다.
* 다음 예제에서는 Basesubscriber의 내부에서 request 메서드를 호출해 원하는 만큼의 데이터를 요청한다. 이를 통해 데이터가 한 번에 모두 Subscriber에 전달되는 것이 아니라, Subscriber가 요청했을 때 데이터가 전달된다.

```java
Flux.range(1, 5)
    .doOnRequest(data -> log.info("# doOnRequest: {}", data))
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(1);
        }

        @SneakyThrows
        @Override
        protected void hookOnNext(Integer value) {
            Thread.sleep(2000L);
            log.info("# hookOnNext: {}", value);
            request(1);
        }
    });
```

### Backpressure 전략

* IGNORE
  * backpressure를 적용하지 않는 전략으로, Downstream에서의 backpressure 요청이 무시된다. 따라서 IllegalStateException이 발생할 수 있다.
*   ERROR

    * downstream의 데이터 처리 속도가 느려서 upstream의 emit 속도를 따라갈 수 없는 경우 버퍼에 데이터가 가득 차 overflow가 발생하여 IllegalStateException을 발생시킨다.

    ```java
    Flux
        .interval(Duration.ofMillis(1L))
        .onBackpressureError()
        .doOnNext(data -> log.info("# doOnNext: {}", data))
        .publishOn(Schedulers.parallel())
        .subscribe(data -> {
                    try {
                        Thread.sleep(5L);
                    } catch (InterruptedException e) {}
                    log.info("# onNext: {}", data);
                },
                error -> log.error("# onError", error));
    ```
* DROP
  * publisher가 downstream으로 전달한 데이터가 버퍼에 가득 찰 경우, 버퍼에 추가되기 위해 대기중인 데이터 중 먼저 emit된 데이터부터 drop(폐기)시키는 전략이다.
* LATEST
  * publisher가 downstream으로 전달할 데이터가 버퍼에 가득찰 경우 버퍼 밖에서 대기 중인 가장 최근 emit 데이터부터 버퍼에 채운다.
  * 버퍼가 비어 데이터를 넣을 수 있는 시점에, 가장 최근의 데이터만 반영된다.
  * 즉, 버퍼가 가득 차있는 상태에서 새로운 데이터가 들어올 때 마다 이전의 데이터는 폐기된다.
* BUFFER
  * 버퍼의 내부 데이터를 폐기하지않고 버퍼링하거나, 버퍼가 가득차면 버퍼 내부 데이터를 폐기하거나 에러를 발생시키는 다양한 전략을 지원한다. 여기서는 DROP\_LATEST, DROP\_OLDEST에 대해서만 다룬다.
  *   DROP\_LATEST

      * publisher가 downstream으로 전달할 데이터가 버퍼에 가득 차 overflow가 발생한 경우 가장 최근에 버퍼에 추가된 데이터를 drop한다.

      ```java
      Flux
          .interval(Duration.ofMillis(300L))
          .doOnNext(data -> log.info("# emitted by original Flux: {}", data))
          .onBackpressureBuffer(2,
                  dropped -> log.info("** Overflow & Dropped: {} **", dropped),
                  BufferOverflowStrategy.DROP_LATEST)
          .doOnNext(data -> log.info("[ # emitted by Buffer: {} ]", data))
          .publishOn(Schedulers.parallel(), false, 1)
          .subscribe(data -> {
                      try {
                          Thread.sleep(1000L);
                      } catch (InterruptedException e) {}
                      log.info("# onNext: {}", data);
                  },
                  error -> log.error("# onError", error));
      ```
  *   DROP\_OLDEST

      * publisher가 downstream으로 전달할 데이터가 버퍼에 가득 차 overflow가 발생한 경우 가장 예전에 버퍼에 추가된 데이터를 drop한다.

      ```java
      Flux
          .interval(Duration.ofMillis(300L))
          .doOnNext(data -> log.info("# emitted by original Flux: {}", data))
          .onBackpressureBuffer(2,
                  dropped -> log.info("** Overflow & Dropped: {} **", dropped),
                  BufferOverflowStrategy.DROP_OLDEST)
          .doOnNext(data -> log.info("[ # emitted by Buffer: {} ]", data))
          .publishOn(Schedulers.parallel(), false, 1)
          .subscribe(data -> {
                      try {
                          Thread.sleep(1000L);
                      } catch (InterruptedException e) {}
                      log.info("# onNext: {}", data);
                  },
                  error -> log.error("# onError", error));
      ```
