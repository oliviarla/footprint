# Cold/Hot Sequence

## Cold Sequence / Hot Sequence

* Sequence란 Publisher가 emit하는 데이터의 연속적인 흐름을 의미한다.
* 보통 `Cold`는 처음부터 시작한다라는 의미를 갖고, `Hot`은 같은 작업을 재시작하지 않고 바로 사용한다는 의미를 가진다.

### Cold Sequence

* Cold Sequence란 Subscriber가 구독할 때 마다 데이터 흐름이 처음부터 시작되는 Sequence이다.
* 즉, 여러 Subscriber의 구독 시점이 다르더라도 모두 맨 처음 데이터부터 받게 된다.
* Cold Publisher란 Cold Sequence 흐름대로 동작하는 Publisher이다.
* 기본적으로 Mono, Flux는 Cold Publisher이다.

```java
Flux<String> flux = Flux.fromIterable(Arrays.asList(...));

flux.subscribe(data -> log.info("# Subscriber1: {}", data);
```

### Hot Sequence

* 구독이 발생한 시점 이후에 emit된 데이터만 전달받을 수 있는 Sequence이다.
* cache() Operator는 Cold Sequence로 동작하는 Mono를 Hot Sequence로 바꿔주고 emit된 데이터를 캐시하여 구독이 발생할 때 마다 캐싱된 데이터를 전달한다. 이를 통해 구독이 발생할 때 마다 동일한 데이터를 전달받는다.
* share() Operator는 기본적으로 Cold Sequence로 동작하는 것을 Hot Sequence로 동작하도록 바꾸어주는 역할을 한다. 최초 구독이 발생했을 때 데이터를 emit한다.

```java
Flux<String> flux = Flux.fromArray(Arrays.asList(...)).share();
```
