# 테스트

## 테스트 방법

* reactor에서는 reactor-test 라는 라이브러리를 제공하여 쉽게 테스트할 수 있도록 한다.

### StepVerifier

* Reactor Sequence를 정의해두고 구독이 발생했을 때 어떤 signal이 발생하는지 원하는 데이터가 emit되었는지 등을 검증할 수 있다.
* expect
  * Reactor Sequence에서 발생하는 signal의 기댓값을 정의하여 테스트한다.
  * 구독이 이루어지는지, signal을 통해 전달되는 값이 주어진 값과 동일한지, onComplete / onError signal이 전송되는지 등을 검증할 수 있다.
*   verify

    * 내부적으로 테스트 대상 operator 체인에 대해 구독을 발생시켜 기댓값을 평가한다.
    * 다음은 가장 기본적인 테스트 코드 형태이다.

    ```java
    Flux<Integer> source = Flux.just(2, 4, 6, 8, 10);
    StepVerifier
            .create(GeneralTestExample.divideByTwo(source))
            .expectSubscription()
            .expectNext(1)
            .expectNext(2)
            .expectNext(3)
            .expectNext(4)
            .expectError()
            .verify();
    ```

    * 지정한 시간 내에 작업이 종료됨을 검증할 수 있다.

    ```java
    StepVerifier
            .create(TimeBasedTestExample.getCOVID19Count(
                            Flux.interval(Duration.ofMinutes(1)).take(1)
                    )
            )
            .expectSubscription()
            .expectNextCount(11)
            .expectComplete()
            .verify(Duration.ofSeconds(3));
    ```
*   시간 기반 테스트

    * withVirtualTime() 메서드를 제공하여, operator의 Sequence가 동작하여 기대하는 상태가 될 때 까지 오랜 시간이 지나야 하는 경우 이를 기다리지 않고 바로 검증할 수 있도록 한다.
    * VirtualTimeScheduler 클래스의 메서드를 이용해 주어진 시간만큼 넘긴 것 처럼 테스트를 실행할 수 있다.

    ```java
    StepVerifier
            .withVirtualTime(() -> TimeBasedTestExample.getCOVID19Count(
                            Flux.interval(Duration.ofHours(1)).take(1)
                    )
            )
            .expectSubscription()
            .then(() -> VirtualTimeScheduler
                                .get()
                                .advanceTimeBy(Duration.ofHours(1)))
            .expectNextCount(11)
            .expectComplete()
            .verify();
    ```
*   backpressure 테스트

    * 오버플로가 발생했을 때 error가 발생하도록 되어 있는 오퍼레이터를 테스트할 때, 에러가 발생하였고 drop 된 데이터가 존재하는지 검증할 수 있다.

    ```java
    StepVerifier
            .create(BackpressureTestExample.generateNumber(), 1L)
            .thenConsumeWhile(num -> num >= 1)
            .expectError()
            .verifyThenAssertThat()
            .hasDroppedElements();
    ```
*   context 테스트

    * context가 전파됨을 확인하고 어떤 키에 대한 값이 존재하는지 검증할 수 있다.

    ```java
    StepVerifier
            .create(
                ContextTestExample
                    .getSecretMessage(source)
                    .contextWrite(context ->
                                    context.put("secretMessage", "Hello, Reactor"))
                    .contextWrite(context -> context.put("secretKey", "aGVsbG8="))
            )
            .expectSubscription()
            .expectAccessibleContext()
            .hasKey("secretKey")
            .hasKey("secretMessage")
            .then()
            .expectNext("Hello, Reactor")
            .expectComplete()
            .verify();
    ```
*   record 기반 테스트

    * emit되는 데이터를 상세하게 검증하고자 할 때 record 기반 테스트를 사용하면 유용하다.
    * recordWith 메서드로 emit된 데이터를 기록하고, consumeRecordWith 혹은 expectRecordedMatches 메서드로 기록된 데이터들을 검증할 수 있다.

    ```java
    StepVerifier
            .create(RecordTestExample.getCapitalizedCountry(
                    Flux.just("korea", "england", "canada", "india")))
            .expectSubscription()
            .recordWith(ArrayList::new)
            .thenConsumeWhile(country -> !country.isEmpty())
            .consumeRecordedWith(countries -> {
                assertThat(
                        countries
                                .stream()
                                .allMatch(country ->
                                        Character.isUpperCase(country.charAt(0))),
                        is(true)
                );
            })
            .expectComplete()
            .verify();
    ```

### TestPublisher

* 직접 signal을 발생시켜 원하는 상황을 재연할 수 있다.
* 복잡한 로직이 포함된 메서드를 테스트하거나 조건에 따라 signal을 변경해야 하는 등의 상황을 테스트하기 용이하다.

```java
TestPublisher<Integer> source = TestPublisher.create();

StepVerifier
        .create(GeneralTestExample.divideByTwo(source.flux()))
        .expectSubscription()
        .then(() -> source.emit(2, 4, 6, 8, 10))
        .expectNext(1, 2, 3, 4)
        .expectError()
        .verify();
```

*   Well-behaved vs Misbehaving TestPublisher

    * 기본적인 방식으로 TestPublisher 생성 시, null을 emit한다거나 요청하는 개수보다 많은 양을 emit하는 등 리액티브 스트림즈 사양을 위반하는지 여부를 signal을 전송하기 전 검증 작업에서 확인한다.
    * 리액티브 스트림즈 사양을 위반하는지를 사전에 검증하지 않고 오동작하는 Misbehaving publisher를 사용하려면 TestPublisher.createNonCompliant 메서드를 이용해 객체를 생성해야 한다.
      * ALLOW\_NULL : null 데이터를 전송하는 것을 허용한다.
      * CLEANUP\_ON\_TERMINATE : terminal signal (onComplete, onError) 을 여러 번 전송하는 것을 허용한다.
      * DEFER\_CANCELLATION : cancel signal을 무시하고 signal을 보내는 것을 허용한다.
      * REQUEST\_OVERFLOW : 요청 개수보다 많은 signal이 발생하도 다음 호출을 진행한다.

    ```java
    TestPublisher<Integer> source = TestPublisher.createNoncompliant(TestPublisher.Violation.ALLOW_NULL);
    ```

### PublisherProbe

* Sequence의 실행 경로를 추적할 수 있다.
* switchIfEmpty 오퍼레이터의 경우 upstream publisher가 데이터 emit 없이 종료될 때 대체 publisher가 데이터를 emit하도록 도와준다. 단, 오퍼레이터 인자를 넘길 때 메서드를 호출하도록 하면, switchIfEmpty 오퍼레이터가 실행되기 전에 해당 메서드가 먼저 실행된다. 이는 추후 다룰 defer 오퍼레이터를 이용해 해결할 수 있다.
* 이 때 어떤 publisher가 동작하는지 확인하기 위해 PublisherProbe를 사용할 수 있다.

```java
public class PublisherProbeTestExample {
  public static Mono<String> processTask(Mono<String> main, Mono<String> standby) {
      return main
              .flatMap(massage -> Mono.just(massage))
              .switchIfEmpty(standby);
  }
}
```

```java
PublisherProbe<String> probe =
        PublisherProbe.of(PublisherProbeTestExample.supplyStandbyPower());

StepVerifier
        .create(PublisherProbeTestExample
                .processTask( // 내부적으로 switchIfEmpty 오퍼레이터를 사용하는 메서드이다.
                        Mono.empty(), // 주 publisher
                        probe.mono()) // 대체 publisher
        )
        .expectNextCount(1)
        .verifyComplete();

// 대체 publisher에 구독, 요청이 발생했고 취소되지 않았는지 검증할 수 있다.
probe.assertWasSubscribed();
probe.assertWasRequested();
probe.assertWasNotCancelled();
```
