# 오퍼레이터

## sequence 생성

* justOrEmpty
  * just()를 확장한 오퍼레이터
  * null을 emit해도 NullPointerException을 던지지 않고 onComplete signal을 전송한다.
* fromIterable
  * Java Iterable의 구현체에 포함된 데이터를 emit하는 Flux를 생성한다.
* fromStream
  * Java Stream에 포함된 데이터를 emit하는 Flux를 생성한다.
  * Stream은 재사용이 불가능하므로 cancel, error, complete 시에 자동으로 닫힌다.
* range
  * 입력된 숫자 n부터 1씩 증가하는 연속된 수를 m개 emit하는 Flux를 생성한다.
  * `Flux.range(n, m)`
*   defer

    * 구독이 발생하는 시점에 데이터를 emit하는 Flux / Mono를 생성한다.
    * 다른 오퍼레이터를 defer 오퍼레이터로 감싸면 필요한 시점에 데이터를 emit하도록 지연시킬 수 있다.
    * just 오퍼레이터의 경우 Hot Publisher이므로 구독 여부와 상관 없이 데이터를 emit한다. 따라서 구독이 발생하면 기존 데이터를 replay해 subscriber에 전달한다.

    ```java
    Mono<LocalDateTime> justMono = Mono.just(LocalDateTime.now()); // 12:00:00
    Mono<LocalDateTime> deferMono = Mono.defer(() ->
                                                Mono.just(LocalDateTime.now()));

    Thread.sleep(2000);

    justMono.subscribe(data -> log.info("# onNext just1: {}", data)); // 12:00:00
    deferMono.subscribe(data -> log.info("# onNext defer1: {}", data)); // 12:00:02

    Thread.sleep(2000);

    justMono.subscribe(data -> log.info("# onNext just2: {}", data)); // 12:00:00
    deferMono.subscribe(data -> log.info("# onNext defer2: {}", data)); // 12:00:04
    ```

    * switchIfEmpty 오퍼레이터 인자로 메서드를 호출하도록 하면, 주 Publisher의 empty 유무에 상관 없이 해당 메서드가 수행된다. 불필요한 수행을 막으려면 defer 오퍼레이터와 메서드 호출을 조합해야 한다.

    ```java
    Mono
        .just("Hello")
        .delayElement(Duration.ofSeconds(3))
        .switchIfEmpty(Mono.defer(() -> sayDefault()))
        .subscribe(data -> log.info("# onNext: {}", data));
    ```
*   using

    * 파라미터로 전달받은 resource의 데이터를 emit하는 Flux를 생성한다.
    * 읽어올 resource, resource를 emit하는 Flux, 종료 signal이 발생 시 리소스를 해제하기 위한 람다식을 입력받는다.

    ```java
    Flux
        .using(() -> Files.lines(path), Flux::fromStream, Stream::close)
        .subscribe(log::info);
    ```
*   generate

    * 동기적으로 한 번에 한 건씩 emit한다.

    ```java
    Flux
        .generate(() -> 0, (state, sink) -> {
            sink.next(state);
            if (state == 10)
                sink.complete();
            return ++state;
        })
        .subscribe(data -> log.info("# onNext: {}", data));
    ```
*   create

    * 한 번에 여러 건의 데이터를 비동기적으로 emit한다.
    * request 메서드로 요청을 받았을 때 publisher가 데이터를 emit하는 pull 방식, subscriber의 요청과 관계 없이 비동기적으로 데이터를 emit하는 push 방식을 사용할 수 있다.
    * pull 방식 예제
      * 구독이 발생하면 hookOnSubscribe 메서드에 의해 2개의 데이터를 request하게 되고, 이를 통해 sink.onRequest 메서드가 수행된다.
      * sink.onRequest 메서드 내부에서는 요청된 데이터 수 만큼 emit한다.

    ```java
    Flux.create((FluxSink<Integer> sink) -> {
        sink.onRequest(n -> {
            try {
                Thread.sleep(1000L);
                for (int i = 0; i < n; i++) {
                    if (COUNT >= 9) {
                        sink.complete();
                    } else {
                        COUNT++;
                        sink.next(DATA_SOURCE.get(COUNT));
                    }
                }
            } catch (InterruptedException e) {}
        });

        sink.onDispose(() -> log.info("# clean up"));
    }).subscribe(new BaseSubscriber<>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(2);
        }

        @Override
        protected void hookOnNext(Integer value) {
            SIZE++;
            log.info("# onNext: {}", value);
            if (SIZE == 2) {
                request(2);
                SIZE = 0;
            }
        }

        @Override
        protected void hookOnComplete() {
            log.info("# onComplete");
        }
    });
    ```

    * push 방식 예제
      * priceEmitter 라는 객체는 가격 데이터가 변동되었을 때 해당 객체에 등록된 Listener의 onPrice 메서드를 호출한다.
      * 이를 통해 subscriber에게 변동되는 가격 데이터를 비동기적으로 emit할 수 있게 된다.

    ```java
    Flux.create((FluxSink<Integer> sink) ->
                    priceEmitter.setListener(new CryptoCurrencyPriceListener() {
        @Override
        public void onPrice(List<Integer> priceList) {
            priceList.stream().forEach(price -> {
                sink.next(price);
            });
        }

        @Override
        public void onComplete() {
            sink.complete();
        }
    }))
    .publishOn(Schedulers.parallel())
    .subscribe(
        data -> log.info("# onNext: {}", data),
        error -> {},
        () -> log.info("# onComplete"));

    Thread.sleep(3000L);

    priceEmitter.flowInto(); // 변동되는 가격을 받아올 수 있도록 한다.

    Thread.sleep(2000L);
    priceEmitter.complete(); // 가격을 더이상 받아올 필요가 없으면 complete 처리한다.
    ```

    * create 오퍼레이터는 한 번에 여러 건의 데이터를 비동기적으로 emit할 수 있으므로 **Backpressure 전략**이 필요하다.
      * 다음 예제의 경우 Request가 들어왔을 때 4개씩 데이터를 emit하는데, publishOn에 의해 2개의 데이터만 보관할 수 있다.
      * 따라서 2개의 데이터만 보존하고 나머지 데이터는 DROP하도록 구현할 수 있다.

    ```java
    int start = 1;
    int end = 4;

    Flux.create((FluxSink<Integer> emitter) -> {
        emitter.onRequest(n -> {
            log.info("# requested: " + n);
            try {
                Thread.sleep(500L);
                for (int i = start; i <= end; i++) {
                    emitter.next(i);
                }
                start += 4;
                end += 4;
            } catch (InterruptedException e) {}
        });

        emitter.onDispose(() -> {
            log.info("# clean up");
        });
    }, FluxSink.OverflowStrategy.DROP) // 오버플로우 발생 시 DROP하도록 한다.
    .subscribeOn(Schedulers.boundedElastic())
    .publishOn(Schedulers.parallel(), 2)
    .subscribe(data -> log.info("# onNext: {}", data));

    ```

#### sequence 필터링

* filter
  * upstream에서 emit된 데이터 중 조건에 일치하는 데이터만 downstream으로 emit한다.
* filterWhen
  * Inner Sequence를 통해 조건에 맞는 데이터를 비동기적으로 필터링을 수행한다.
* skip
  * upstream에서 emit된 데이터 중 일정량의 데이터 혹은 시간만큼 건너뛴 후 나머지 데이터를 downstream으로 emit한다.
* take
  * upstream에서 emit된 데이터 중 일정량의 데이터만 혹은 일정 시간동안 emit된 데이터만 downstream으로 emit한다.
* takeLast
  * upstream에서 emit된 데이터 중 마지막 데이터부터 역순으로 파라미터로 입력한 개수만큼 downstream으로 emit한다.
* takeUntil
  * 파라미터로 입력한 람다 표현식이 true가 되기 전까지 upstream에서 emit된 데이터를 downstream으로 emit한다.
  * 람다 표현식이 true가 되었을 때 해당 데이터까지 emit되고, 다음 데이터부터 emit되지 않음을 주의해야 한다.
* takeWhile
  * 파라미터로 입력한 람다 표현식이 true인 동안에만 upstream에서 emit된 데이터를 downstream으로 emit한다.
  * 람다 표현식이 false가 되었을 때 해당 데이터는 emit되지 않고 sequence가 종료된다.
* next
  * upstream에서 emit되는 데이터 중 첫 데이터만 downstream으로 emit한다.
  * 만약 첫 데이터가 empty라면 downstream으로 Mono.empty()를 반환한다.

#### Sequence 변환

* map
  * upstream에서 emit된 데이터를 mapper function을 통해 변환한 후 downstream으로 emit한다.
  * 오퍼레이터 내부에서 에러 발생 시 sequence가 종료되지 않고 계속 진행하도록 하는 기능을 제공한다.
*   flatMap

    * upstream에서 emit된 데이터를 inner sequence에서 다른 데이터와 합치는 평탄화(flatten) 작업을 통해 하나의 sequence로 병합한다.

    ```java
    ```

    * inner sequence에서 Scheduler를 설정해 비동기적으로 데이터를 emit하도록 하면 downstream으로 emit하는 데이터 순서를 보장하지 않는다.

    ```java
    ```
* concat
  * 여러 publisher의 sequence를 연결해 데이터를 순차적으로 emit한다.
  * 먼저 입력된 publisher의 sequence가 종료될 때까지 다른 publisher의 sequence는 대기한다.
* merge
  * publisher의 sequence에서 emit된 데이터를 interleave 방식으로 병합한다.
  * 모든 publisher의 sequence가 동시에 subscribe되어 먼저 emit된 순서대로 downstream에 전달된다.
*   zip

    * 각 publisher sequence에서 emit된 데이터를 하나의 튜플로 결합한다.
    * 여러 publisher sequence에서 데이터가 emit되는 시간이 다르다면 다른 publisher sequence에서 데이터가 emit되어 한 쌍을 만들 수 있을 때까지 기다린다.

    ```java
    ```

    * combinator를 인자로 전달해 여러 publisher sequence에서 emit된 데이터를 변환해 downstream에 전달할 수도 있다.

    ```java
    ```
* and
  * Mono의 complete 시그널과 Publisher의 complete 시그널을 결합해 Mono\<Void>를 반환한다.
  * 따라서 upstream에서 emit된 데이터는 전달되지 않는다.
  * 여러 작업이 모두 끝난 시점에 최종적으로 작업을 수행하기에 용이하다.
* collectList
  * Flux에서 emit된 데이터를 모아 List로 변환 후, 해당 List를 emit하는 Mono를 반환한다.
  * upstream sequence가 비어있다면 empty list를 반환한다.
* collectMap
  * Flux에서 emit된 데이터를 기반으로 key, value를 생성해 Map을 구성하고, 해당 Map을 emit하는 Mono를 반환한다.

#### Sequence 내부 동작 확인

* emit되는 데이터 변경 없이 side effect만을 수행하기 위한 오퍼레이터들이 존재한다.
* doOnXXX() 오퍼레이터들은 Consumer 혹은 Runnable 타입을 인자로 받아 내부 동작 상의 데이터/에러를 확인하거나 로그를 출력하는 등 주로 디버깅 용도로 사용된다.

#### 에러 처리

*   error

    * 특정 에러를 던지며 종료하는 Flux를 생성한다.
    * 어떠한 sequence에서 checked exception이 던져졌을 때, 이를 다시 unchecked exception으로 던져onError signal 형태로 downstream에 전달하고자 할 때 사용할 수 있다.

    ```java
    14-44
    ```
* onErrorReturn
  * upstream에서 에러가 발생했을 때 에러 대신 대체 값을 downstream으로 emit한다.
  * 모든 예외 타입에 대해 대체 값을 설정하는 것이 기본적이고, 특정 예외 타입에 한해서만 대체 값을 설정할 수도 있다.
*   onErrorResume

    * 에러가 발생했을 때 에러 대신 대체 publisher를 반환한다.
    * 캐시에서 데이터를 검색하였을 때 데이터가 없다면 데이터베이스로부터 찾도록 할 때 유용하다.

    ```java
    14-47
    ```
* onErrorContinue
  * 에러가 발생했을 때 에러가 발생했던 데이터는 인자로 입력받은 BiConsumer에 의해 로깅 등의 작업을 수행하도록 하고, upstream에서는 아무 영향 없이 다음 데이터를 emit할 수 있도록 한다. 따라서 에러가 발생하더라도 sequence가 종료되지 않고 계속 이어지도록 한다.
  * 공식 문서에서는 onErrorContinue 오퍼레이터가 명확하지 않은 sequence 동작으로 개발자가 의도하지 않은 상황을 발생시킬 수 있으므로, doOnError를 통해 로그를 기록하고 onErrorResume 오퍼레이터로 작업하도록 권고한다.
*   retry

    * publisher가 데이터를 emit하는 과정에서 에러가 발생하면 특정 횟수만큼 원본 Flux의 sequence를 다시 구독한다.
    * timeout 오퍼레이터와 함께 사용하면, 네트워크 지연으로 인해 타임아웃이 발생해서 재요청해야 하는 상황에서 유용하다.
    * 재시도로 인해 발생한 데이터 중복을 제거하기 위해 `collect(Collectors.toSet())` 을 사용할 수 있다.

    ```java
    Flux
        .fromIterable(SampleData.books)
        .delayElements(Duration.ofMillis(500))
        .map(book -> {
            try {
                count[0]++;
                if (count[0] == 3) {
                    Thread.sleep(2000);
                }
            } catch (InterruptedException e) {
            }

            return book;
        })
        .timeout(Duration.ofSeconds(2))
        .retry(1)
        .doOnNext(book -> log.info("# getBooks > doOnNext: {}, price: {}",
                book.getBookName(), book.getPrice()))
        .collect(Collectors.toSet()) // emit된 데이터들을 모은 후 중복 제거
        .subscribe(bookSet -> bookSet.stream()
                .forEach(book -> log.info("book name: {}, price: {}",
                        book.getBookName(), book.getPrice())));
    ```

#### Sequence 시간 측정

* elapsed
  * emit된 데이터 사이의 경과 시간을 측정해 `Tuple<Long, T>` 형태로 downstream에 emit한다.
  * 가장 처음 emit되는 데이터의 경우 onSubscribe signal이 발생한 시점과 데이터가 emit된 시점을 기준으로 시간이 측정된다.
  * 측정 시간은 ms 단위이다.

#### Flux Sequence 분할

*   window

    * upstream에서 emit되는 데이터를 일정 개수(maxSize)만큼만 포함하는 새로운 Flux로 분할한다. 이렇게 분할된 Flux를 window라고 지칭한다.
    * `사용자가 upstream으로 요청하는 개수 * maxSize` 만큼 upstream에 요청하고, 데이터는 maxSize씩 윈도우로 분할된다.

    ```java
    14-53
    ```
* buffer
  * upstream에서 emit되는 데이터를 일정 개수(maxSize)만큼 List 버퍼로 한번에 emit한다.
  * 높은 처리량을 요구하는 애플리케이션이 있다면 들어오는 데이터를 버퍼링한 후 batch insert 등의 일괄 작업으로 처리하여 성능 향상을 시킬 수 있다.
* bufferTimeout
  * buffer 오퍼레이터와 유사하지만 일정 개수(maxSize)가 될 때까지 계속 기다리는 것이 아니라, timeout을 두어 시간이 만료되면 즉시 List 버퍼에 emit한다.
  * 따라서 항상 maxSize만큼의 List 버퍼가 반환됨을 보장하지 않는다.
*   groupBy

    * emit되는 데이터들을 keyMapper로 생성한 key를 기준으로 그룹화하여 GroupedFlux를 반환한다.
    * valueMapper를 사용할 경우 그룹화하는 과정에서 데이터를 가공할 수 있다.

    ```java
    14-57
    ```

#### Flux를 다수의 Subscriber에게 Multicasting

*   publish

    * 구독이 발생한 시점에 즉시 데이터를 emit하지 않고, connect()를 호출하는 시점에 데이터를 emit한다.
    * Flux는 hot sequence로 변환되어 구독 시점 이후에 emit된 데이터만 전달받을 수 있다.

    ```java
    14-60
    ```

    * autuConnect()와 함께 사용하는 경우 직접 조건을 걸어 connect()가 호출되도록 하지 않고, 특정 수 만큼의 구독이 발생하였을 때 자동으로 데이터가 emit되도록 할 수 있다.

    ```java
    14-62
    ```

    * 구독이 취소되더라도 upstream으로의 연결이 해제되지 않으므로 다른 구독이 새로 발생하더라도 기존 데이터를 이어서 전달받게 된다.

    ```java
    Flux<Long> publisher =
            Flux
                .interval(Duration.ofMillis(500))
                .publish().autoConnect(1);
    Disposable disposable = publisher.subscribe(data -> log.info("# subscriber 1: {}", data));

    Thread.sleep(2100L);
    disposable.dispose(); // 구독 취소

    publisher.subscribe(data -> log.info("# subscriber 2: {}", data));
    ```
*   refCount

    * 파라미터로 입력된 숫자만큼의 구독이 발생하는 시점에 upstream에 연결되고, 모든 구독이 취소되거나 upsteam에서 데이터 emit이 종료되면 연결이 해제된다.
    * 무한 스트림 상황에서 모든 구독이 취소될 경우 연결을 해제시키고자 할 때 유용하다.

    ```java
    ```
