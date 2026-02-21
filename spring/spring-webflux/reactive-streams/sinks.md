# Sinks

## 개념

* Flux, Mono가 onNext 같은 Signal을 내부적으로 전송해준다면, Sinks는 Signal을 명시적으로 전송할 수 있도록 해준다.
* 기존에 존재하던 generate() / create() 오퍼레이터는 싱글 스레드 기반으로 signal을 전송하는 반면, Sinks는 멀티 스레드 방식으로 Signal을 전송하면서 스레드 안전성을 보장한다.
* Sinks에서는 여러 스레드에서 하나의 공유 변수에 동시 접근할 때를 감지하여, 단 하나의 스레드만 접근 가능하도록 하고 나머지는 접근 실패로 처리한다.
* 다음은 create 오퍼레이터를 이용해 signal을 전송하고, doTask 메서드 처리, map 오퍼레이터 처리, Subscriber에서 전달받은 데이터 처리 작업을 모두 각각의 스레드에서 처리하도록 하는 예제이다.

```java
public static void main(String[] args) throws InterruptedException {
    int tasks = 6;
    Flux
        .create((FluxSink<String> sink) -> {
            IntStream
                    .range(1, tasks)
                    .forEach(n -> sink.next(doTask(n)));
        })
        .subscribeOn(Schedulers.boundedElastic())
        .doOnNext(n -> log.info("# create(): {}", n))
        .publishOn(Schedulers.parallel())
        .map(result -> result + " success!")
        .doOnNext(n -> log.info("# map(): {}", n))
        .publishOn(Schedulers.parallel())
        .subscribe(data -> log.info("# onNext: {}", data));

    Thread.sleep(500L);
}

private static String doTask(int taskNumber) {
    // now tasking.
    // complete to task.
    return "task " + taskNumber + " result";
}
```

* doTask 메서드가 여러 스레드에서 처리되도록 하려면 Sinks를 사용해야 한다.
* 아래는 doTask 메서드를 매번 새로운 스레드에서 처리하도록 하고, Sinks를 통해 Downstream에 emit하도록 하는 예제이다.

```java
public static void main(String[] args) throws InterruptedException {
    int tasks = 6;

    Sinks.Many<String> unicastSink = Sinks.many().unicast().onBackpressureBuffer();
    Flux<String> fluxView = unicastSink.asFlux();
    IntStream
            .range(1, tasks)
            .forEach(n -> {
                try {
                    new Thread(() -> {
                        unicastSink.emitNext(doTask(n), Sinks.EmitFailureHandler.FAIL_FAST);
                        log.info("# emitted: {}", n);
                    }).start();
                    Thread.sleep(100L);
                } catch (InterruptedException e) {
                    log.error(e.getMessage());
                }
            });

    fluxView
            .publishOn(Schedulers.parallel())
            .map(result -> result + " success!")
            .doOnNext(n -> log.info("# map(): {}", n))
            .publishOn(Schedulers.parallel())
            .subscribe(data -> log.info("# onNext: {}", data));

    Thread.sleep(200L);
}

private static String doTask(int taskNumber) {
    // now tasking.
    // complete to task.
    return "task " + taskNumber + " result";
}
```

## 종류

### Sinks.One

* 한 건의 데이터를 전송하는 것이 목적
* FAIL\_FAST는 emit 도중 에러가 발생하면 즉시 실패 처리를 하는 EmitFailureHandler를 의미한다.
* Sinks.One 객체를 Mono 객체로 변환하여 데이터를 전달받을 수 있다.
* 아무리 많은 데이터를 emit하더라도 처음 emit한 데이터만 정상적으로 emit되고, 나머지는 drop된다.

```java
Sinks.One<String> sinkOne = Sinks.one();
Mono<String> mono = sinkOne.asMono();

sinkOne.emitValue("Hello Reactor", FAIL_FAST); // emit succeed
sinkOne.emitValue("Hi Reactor", FAIL_FAST); // drop
sinkOne.emitValue(null, FAIL_FAST); // drop

mono.subscribe(data -> log.info("# Subscriber1 {}", data));
mono.subscribe(data -> log.info("# Subscriber2 {}", data));
```

### Sinks.Many

* 여러 건의 데이터를 여러 방식으로 전송하는 것이 목적
* Sinks가 Publisher의 역할을 하는 경우 기본적으로 Hot Publisher로 동작한다.
* ManySpec에서 정의하는 기능들은 다음과 같다.
  *   UnicastSpec : 하나의 Subscriber에게만 데이터를 emit

      ```java
      Sinks.Many<Integer> unicastSink = Sinks.many().unicast().onBackpressureBuffer();
      Flux<Integer> fluxView = unicastSink.asFlux();

      unicastSink.emitNext(1, FAIL_FAST);
      unicastSink.emitNext(2, FAIL_FAST);

      fluxView.subscribe(data -> log.info("# Subscriber1: {}", data)); // 1 2 3

      unicastSink.emitNext(3, FAIL_FAST);

      fluxView.subscribe(data -> log.info("# Subscriber2: {}", data)); // exception 발생
      ```
  *   MulticastSpec : 하나 이상의 Subscriber들에게 데이터를 emit

      ```java
      Sinks.Many<Integer> multicastSink =
              Sinks.many().multicast().onBackpressureBuffer();
      Flux<Integer> fluxView = multicastSink.asFlux();

      multicastSink.emitNext(1, FAIL_FAST);
      multicastSink.emitNext(2, FAIL_FAST);

      fluxView.subscribe(data -> log.info("# Subscriber1: {}", data)); // 1 2 3
      fluxView.subscribe(data -> log.info("# Subscriber2: {}", data)); // 3

      multicastSink.emitNext(3, FAIL_FAST);
      ```
  *   MulticastReplaySpec : Subscriber들에게 특정 시점으로 되돌린 데이터부터 emit

      * 다음 예시는 limit 메서드를 통해 emit된 데이터 중 2건만 뒤로 돌린 후 replay하여 전달하는 예시이다.

      ```java
      Sinks.Many<Integer> replaySink = Sinks.many().replay().limit(2);
      Flux<Integer> fluxView = replaySink.asFlux();

      replaySink.emitNext(1, FAIL_FAST);
      replaySink.emitNext(2, FAIL_FAST);
      replaySink.emitNext(3, FAIL_FAST);

      fluxView.subscribe(data -> log.info("# Subscriber1: {}", data)); // 2,3,4

      replaySink.emitNext(4, FAIL_FAST);

      fluxView.subscribe(data -> log.info("# Subscriber2: {}", data)); // 3,4
      ```
