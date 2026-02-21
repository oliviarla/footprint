# 디버깅

## 디버그의 어려움

* Reactor 작업은 대부분 비동기로 실행되고, Reactor Sequence는 선언형 프로그래밍 방식으로 구성되므로 스택트레이스를 바로 확인하거나 브레이크 포인트를 걸어 디버깅하기 어렵다.
* 이러한 디버깅의 어려움을 최소화하기 위해 몇 가지 방법이 제공된다.

## 디버그 모드

* Hooks.onOperatorDebug() 를 통해 디버그 모드를 활성화하여 에러를 출력할 수 있다.
* 애플리케이션 내에 있는 모든 오퍼레이터의 스택트레이스를 캡쳐하고, 해당 정보를 기반으로 에러가 발생한 Assembly의 스택트레이스를 원본 스택트레이스의 중간에 끼워넣는다.

> assembly란 Operator가 반환하는 새로운 Mono/Flux 객체가 선언된 지점을 의미한다. traceback이란 에러가 발생한 오퍼레이터의 스택트레이스를 캡처한 assembly 정보를 의미한다.

* Rector에서는 모든 오퍼레이터 스택트레이스를 캡쳐하지 않고도 디버깅 정보를 추가할 수 있는 reactor-tools 라이브러리를 제공한다. 프로덕션 환경에서는 이를 추가하여 ReactorDebugAgent를 활성화하면 된다.

## checkpoint 오퍼레이터

* 이 오퍼레이터는 특정 오퍼레이터 체인 내의 스택트레이스만 캡쳐한다.
* 실제 에러가 발생했거나 전파된 assembly 지점의 traceback이 출력된다.

```java
Flux
	  .just(2, 4, 6, 8)
	  .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
	  .map(num -> num + 2)
	  .checkpoint()
	  .subscribe(
	          data -> log.info("# onNext: {}", data),
	          error -> log.error("# onError:", error)
	  );
```

* traceback 출력을 하지 않고 지정된 description만 출력할 수도 있고, traceback과 description을 모두 출력할 수도 있다.

```java
Flux
    .just(2, 4, 6, 8)
    .zipWith(Flux.just(1, 2, 3, 0), (x, y) -> x/y)
    .checkpoint("Example12_4.zipWith.checkpoint") // description 출력
    .map(num -> num + 2)
    .checkpoint("Example12_4.map.checkpoint", true) // traceback도 포함
    .subscribe(
            data -> log.info("# onNext: {}", data),
            error -> log.error("# onError:", error)
    );
```

* 오퍼레이터 체인이 메서드 분리로 인해 기능별로 흩어져 있는 경우 어느 부분에서 에러가 발생했는지 확인하기 어려운데, checkpoint 오퍼레이터를 각 오퍼레이터 체인에 추가한 후 범위를 좁혀가며 디버깅할 수 있다.

## log 오퍼레이터

* Reactor Sequence 동작을 로그로 출력한다. 즉, onSubscribe, onNext, request 등이 어떤 데이터를 기반으로 호출되었는지 출력된다.
* 에러가 발생한 지점에 단계적으로 접근할 수 있다.
* 사용 개수에 제한이 없어 여러 개의 log 오퍼레이터를 두고 내부 동작을 상세히 분석하면서 디버깅할 수 있다.
* 로그 출력 시 사용할 description과 로그 레벨을 지정할 수 있다.

```java
Flux.fromArray(new String[]{"BANANAS", "APPLES", "PEARS", "MELONS"})
        .map(String::toLowerCase)
        .map(fruit -> fruit.substring(0, fruit.length() - 1))
        .log()
				// .log("Fruit.Substring", Level.FINE)
        .map(fruits::get)
        .subscribe(
                log::info,
                error -> log.error("# onError:", error));
```
