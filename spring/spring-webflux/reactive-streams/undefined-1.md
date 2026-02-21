# 컨텍스트

## 컨텍스트란?

* 어떠한 상황에서 그 상황을 처리하기 위해 필요한 정보를 의미한다.
* Reactor에서 컨텍스트란 구성요소 간에 전파되는 key-value 형태의 저장소를 의미한다.
* 인증 정보 등과 같은 **독립성을 가진 정보를 전송**하는 데에 적합하다.

## 사용 방법

*   **operator 체인의 downstream에서 upstream으로 컨텍스트가 전파**되어 각 오퍼레이터가 모두 해당 컨텍스트 정보를 이용할 수 있다.

    ```java
    Mono
        .deferContextual(ctx ->
            Mono.just(ctx.get(key1))
        )
        .publishOn(Schedulers.parallel())
        .contextWrite(context -> context.put(key2, "Bill"))
        .transformDeferredContextual((mono, ctx) ->
                mono.map(data -> data + ", " + ctx.getOrDefault(key2, "Steve")) // key2에 해당하는 데이터가 아직 없음
        )
        .contextWrite(context -> context.put(key1, "Apple"))
        .subscribe(data -> log.info("# onNext: {}", data));
    ```
* 모든 오퍼레이터가 컨텍스트를 읽도록 하려면 오퍼레이터의 맨 마지막에 contextWrite를 해주어야 한다.
* 구독이 발생할 때 마다 해당 구독과 연결된 컨텍스트가 새로 생성된다.
* 동일한 키에 데이터를 중복해서 저장 시, Operator 체인의 가장 윗쪽에서 저장한 값이 최종 저장된다.
* 컨텍스트에 데이터를 쓰려면 contextWrite 오퍼레이터를 사용하면 된다.
  * Context 객체의 put() 메서드를 이용해 데이터를 쓰는 코드를 구현할 수 있다.
  * put 메서드를 통해 데이터를 쓴 후에는 불변 객체를 다음 오퍼레이터로 전달하여 스레드 안전성을 보장한다.
* 컨텍스트의 데이터를 읽으려면 1. deferContextual 오퍼레이터를 사용해 **원본 데이터 소스 레벨에서 읽거나** 2. transformDeferredContextual 오퍼레이터를 이용해 **오퍼레이터 체인 중간에서 읽을** 수 있다.
  * ContextView 객체의 get() 메서드를 통해 데이터를 읽을 수 있다.

```java
Mono
    .deferContextual(ctx ->
        Mono
            .just("Hello" + " " + ctx.get("firstName"))
            .doOnNext(data -> log.info("# just doOnNext : {}", data))
    )
    .subscribeOn(Schedulers.boundedElastic())
    .publishOn(Schedulers.parallel())
    .transformDeferredContextual(
            (mono, ctx) -> mono.map(data -> data + " " + ctx.get("lastName"))
    )
    .contextWrite(context -> context.put("lastName", "Jobs"))
    .contextWrite(context -> context.put("firstName", "Steve"))
    .subscribe(data -> log.info("# onNext: {}", data));
```

* Inner Sequence 외부에서는 Inner Sequence 내부 context에 저장된 데이터에 접근할 수 없다.
* Inner Sequence 내부에서는 외부 context의 데이터에 접근할 수 있다.

> Inner Sequence란 오퍼레이터 내부에 있는 sequence를 의미한다.

```java
String key1 = "company";
Mono
    .just("Steve")
    .transformDeferredContextual((stringMono, ctx) ->
            ctx.get("role")) // 접근에 실패한다.
    .flatMap(name ->
        Mono.deferContextual(ctx ->
            Mono
                .just(ctx.get(key1) + ", " + name) // 접근에 성공한다.
                .transformDeferredContextual((mono, innerCtx) ->
                        mono.map(data -> data + ", " + innerCtx.get("role"))
                )
                .contextWrite(context -> context.put("role", "CEO"))
        )
    )
    .publishOn(Schedulers.parallel())
    .contextWrite(context -> context.put(key1, "Apple"))
    .subscribe(data -> log.info("# onNext: {}", data));
```
