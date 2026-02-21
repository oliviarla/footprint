# 스케줄러

## 스케줄러 역할

* Reactor Sequence에서 사용되는 스레드를 관리한다.
* 스케줄러 전용 operator의 파라미터로 적절한 스케줄러를 전달하면 해당 스케줄러의 특성에 맞는 스레드가 Reactor Sequence에 할당된다.

## 스케줄러 전용 operator

### subscribeOn()

* 구독이 발생한 직후 원본 Publisher 동작을 실행할 스레드를 지정한다.
* 원본 Publisher는 데이터 소스를 emit한다.
* 아래 예시에서 사용된 doOnSubscribe 오퍼레이터는 구독이 발생한 시점에 추가적인 작업이 필요할 때 사용된다.

```java
Flux.fromArray(new Integer[] {1, 3, 5, 7})
        .subscribeOn(Schedulers.boundedElastic())
        .doOnNext(data -> log.info("# doOnNext: {}", data))
        .doOnSubscribe(subscription -> log.info("# doOnSubscribe"))
        .subscribe(data -> log.info("# onNext: {}", data));
```

### publishOn()

* downstream으로 signal을 전송할 때 실행되는 스레드를 제어한다.
* publishOn()을 기준으로 아래쪽에 있는 downstream의 실행 스레드를 지정한다.
* doOnNext 작업 역시 이 때 지정된 스레드에서 수행하게 된다.

```java
Flux.fromArray(new Integer[] {1, 3, 5, 7})
        .doOnNext(data -> log.info("# doOnNext: {}", data))
        .doOnSubscribe(subscription -> log.info("# doOnSubscribe"))
        .publishOn(Schedulers.parallel())
        .subscribe(data -> log.info("# onNext: {}", data)); // parallel 스레드
```

* 한 개 이상 사용할 수 있어 여러 오퍼레이터 체인이 있을 때 각각 다른 스레드에서 실행되도록 할 수 있다.
* prefetch
  * 스케줄러가 생성하는 스레드의 비동기 경계 시점에 미리 보관할 데이터 개수를 지정할 수 있다.
  * publishOn()이 호출될 때 upstream에 n개를 미리 요청해 데이터를 가져오게 된다.

### parallel() + runOn()

* 병렬성을 갖는 물리적 스레드를 제공하기 위해, 라운드로빈으로 물리적 스레드 개수만큼 스레드를 병렬로 실행한다.
* 원하는 만큼 물리적 스레드 개수를 지정할 수도 있다.
* parallel 오퍼레이터는 emit되는 데이터를 물리적 스레드 수에 맞게 골고루 분배하는 역할을 하고, runOn 오퍼레이터는 실제 스레드에 할당해 병렬 작업을 수행한다.
* 라운드 로빈 방식으로 CPU의 논리적인 코어 수에 맞게 데이터를 그룹화한 것을 **rail**이라 한다.
  * 논리적인 코어 개수 == 물리적인 스레드 개수
* 물리적인 코어 개수는 정해져있지만 논리적인 코어 개수는 CPU 하이퍼스레딩 등의 기술에 따라 달라진다.

```java
Flux.fromArray(new Integer[]{1, 3, 5, 7, 9, 11, 13, 15, 17, 19})
        .parallel(4)
        .runOn(Schedulers.parallel())
        .subscribe(data -> log.info("# onNext: {}", data));
```

* subscribeOn 오퍼레이터와 publishOn 오퍼레이터를 함께 사용하면, 원본 Publisher에서 데이터를 emit하는 스레드와 emit된 데이터를 가공하는 스레드를 적절히 분리할 수 있다.
* 아래는 fromArray, filter 오퍼레이터는 boundedElastic 스레드에 의해 수행되도록 하고, map, subscribe 오퍼레이터는 parallel 스레드에 의해 수행되도록 한 예제이다.

```java
Flux
    .fromArray(new Integer[] {1, 3, 5, 7})
    .subscribeOn(Schedulers.boundedElastic())
    .doOnNext(data -> log.info("# doOnNext fromArray: {}", data))
    .filter(data -> data > 3)
    .doOnNext(data -> log.info("# doOnNext filter: {}", data))
    .publishOn(Schedulers.parallel())
    .map(data -> data * 10)
    .doOnNext(data -> log.info("# doOnNext map: {}", data))
    .subscribe(data -> log.info("# onNext: {}", data));
```

## 스케줄러 종류

* immediate()
  * 별도 스레드를 생성하지 않고 현재 스레드에서 작업을 처리할 때 사용한다.
* single()
  * 하나의 스레드만 생성해 스케줄러가 제거되기 전까지 재사용한다.
  * 지연 시간이 짧은 작업을 처리하는 목적으로 사용된다.
* newSingle()
  * 호출할 때 마다 새로운 스레드 하나를 생성해 사용한다.
  * 데몬스레드 여부를 지정할 수 있다.
* boundedElastic()
  * ExecutorService 기반의 스레드 풀을 생성해 정해진 수 만큼의 스레드를 이용해 작업을 처리한다.
  * 작업이 완료된 스레드는 스레드 풀에 반납한다.
  * 기본적으로 CPU 코어 개수 x 10 만큼의 스레드를 생성해두고, 모든 스레드가 사용중이라면 최대 10만개의 작업이 큐에서 대기할 수 있다.
  * blocking I/O 작업을 처리하는 목적으로 사용된다.
* parallel()
  * CPU 코어 개수만큼 스레드를 생성한다.
  * non blocking I/O 작업을 처리하는 목적으로 사용된다.
* fromExecutorService
  * 기존에 사용중인 ExecutorService로부터 스케줄러를 생성하는 방식이다.
* newXXXX()
  * new가 붙지 않은 스케줄러 메서드들은 Reactor에서 제공하는 기본 스케줄러 객체를 사용한다.
  * newSingle, newBoundedElastic, newParallel 메서드를 사용할 경우 새로운 스케줄러 객체를 생성해 사용한다.
  * 커스텀한 스레드 풀을 생성해 사용할 수 있다.
