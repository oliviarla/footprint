# 16장: CompletableFuture

## 🏝️ Future의 단순 활용

* Java 5 부터 미래의 어느 시점에 결과를 얻는 모델에 활용할수 있도록 Future 인터페이스를 제공하고 있다.
* 시간이 걸리는 작업들을 Future 내부에 두면, 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업들을 할 수 있다.
* 아래 코드는 Future로 다른 스레드에 오래걸리는 작업을 할당하고, 호출자 스레드에서는 future.get() 하기 전 로직을 수행하여 비동기적으로 동작한다.

```java
ExecutorService executor = Executors.newCachedThreadPool();

Future<Double> future = executor.submit(new Callable<Double>() {
  public Double call() {
    return doSomeLongComputation();
  }
})

doSomeThingElse();

try {
  future.get(1, TimeUnit.SECONDS); // 비동기 작업의 결과가 준비되지 않았다면 1초까지만 기다려본다.
} catch (ExecutionException ee) {
  // 계산 중 예외 발생
} catch (InterruptedException ie) {
  // 현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException te) {
  // Future가 완료되기 전 타임아웃 발생
}
```

<figure><img src="../../.gitbook/assets/image (163).png" alt="" width="563"><figcaption></figcaption></figure>

* 여러 Future의 결과가 있을 때 의존성을 표현하기 어렵다. A의 계산이 끝나면 B에게 그 결과를 전달해 계산하고, B의 계산까지 끝나면 다른 질의의 결과와 조합하라는 요구사항이 있는 경우 Future로는 구현이 어렵다.
*   CompletableFuture 클래스는 아래와 같은 기능을 제공한다.

    * 두 비동기 계산 결과를 하나로 합칠 수 있다. 서로 독립적일수도, 의존적일수도 있다.
    * Future 집합이 실행하는 모든 태스크의 완료를 기다린다.
    * Future 집합에서 가장 빨리 완료되는 태스크를 기다렸다가 결과를 얻는다.
    * 프로그램적으로 Future를 완료시킨다.(비동기 동작에 수동으로 결과 제공)
    * Future 완료 동작에 반응한다.(결과를 기다리면서 블록되지 않고 결과가 준비되었다는 알림을 받은 후 Future의 결과에 원하는 추가 동작을 수행할 수 있다)



&#x20;

## 🏝️ 비동기 API 구현

### 동기 메서드를 비동기 메서드로 변환

* 상품의 가격 정보를 반환하는 메서드를 비동기 API로 구현한다.
* DB 접근, 할인 서비스 호출 등의 작업을 통해 상품의 가격을 얻는 것 처럼 calculatePrice 메서드를 작성했다.

```java
public double getPrice(String product) {
  return calculatePrice(product);
}

private double calculatePrice(String product) {
  delay(); //1초간 블록
  return random.nextDouble() * product.charAt(0) + product.charAt(1);
}
```

* 위 메서드를 블로킹되지 않도록 하기 위해 Future를 반환한다.

```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();
  new Thread(() -> {
    double price = calculatePrice(product);
    futurePrice.complete(price);
  }).start();
  return futurePrice;
}
```

* 아래와 같이 제품의 가격 정보를 요청보내고 future 결과가 돌아오기 전까지 다른 작업을 처리할 수 있다.

```java
Shop shop = new Shop("BestShop");
long start = System.nanoTime();
Future<Double> futurePrice = shop.getPriceAsync("my favorite product"); //제품 가격 요청
long invocationTime = ((System.nanoTime() - start) / 1_000_000);

//제품의 가격을 계산하는 동안 다른 상점 검색 등 작업 수행
doSomethingElse();

try {
  double price = future.get(); //가격정보를 받을때까지 블록
} catch (Exception e) {
  throw new RuntimeException(e);
}
long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
```

### 에러 처리 방법

* 예외 발생 시 해당 스레드에만 영향을 미친다.
* 따라서 에러가 발생해도 클라이언트는 get 메서드가 반환될 때를 기다리며 영원히 블로킹된다.
* 이를 해결하기 위해 get 메서드에 타임아웃값을 넣어주고, CompletableFuture 내부에서 발생된 에러 정보를 포함시키는 completeExceptionally 메서드를 사용해 외부로 예외를 전달한다.
* 전달된 예외는 ExecutionException에 감싸서 던져진다.

```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();
  new Thread(() -> {
    try {
      double price = calculatePrice(product);
      futurePrice.complete(price);
    } catch (Exception e) {
      futurePrice.completeExceptionally(ex); //에러를 포함시켜 Future를 종료
    }
  }).start();
  return futurePrice;
}
```

### supplyAsync

* CompletableFuture를 직접 생성하는 대신 supplyAsync() 메서드를 이용해 생성되도록 할 수 있다.
* Supplier를 인수로 받아 CompletableFuture를 반환하는 메서드
* ForkJoinPool의 Executor 중 하나가 Supplier를 실행하며, ForkJoinPool의 Executor 대신 다른 Executor를 지정하려면 두 번째 인자로 넣어주면 된다.

```java
public Future<Double> getPriceAsync(String product) {
  return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

## 🏝️ 비블록 코드

* 블록 메서드를 사용할 수밖에 없는 상황에서 비동기적으로 여러 API를 호출하여 프로그램 성능을 높일 수 있도록 하자.
* 아래와 같이 각각의 shop에 존재하는 product의 가격을 가져오도록 하는 메서드가 있다고 한다. 이 때 하나의 가게에서 상품 가격을 가져오는 작업이 약 1초가 걸린다면 \<shop의 개수>초 동안 스레드가 블로킹된다.
* 이 코드를 점차 비블록 코드로 개선해보자.

```java
public List<String> findPrices(String product) {
  return shops.stream()
    .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
    .collect(toList());
}
```

### 병렬 스트림

* 순차 계산을 병렬로 처리해 성능을 개선할 수 있다.
* shop에서 product 가격을 가져오는 작업들이 병렬로 처리되므로 1초 남짓의 시간에 모든 작업이 완료될 것이다.

```java
public List<String> findPrices(String product) {
  return shops.parallelStream()
    .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
    .collect(toList());
}
```

### CompletableFuture

* CompletableFuture로 가게의 상품 가격을 찾는 로직을 비동기적으로 수행한다.
* 스트림 연산은 lazy하기 때문에 하나의 파이프라인(map)으로 연산을 처리하면 모든 가격 정보 요청 동작을 동기적으로 수행하게 된다.
* 즉, 가게의 상품 가격을 하나 찾으면 join을 호출하고, 또 다른 가게의 상품 가격을 찾으면 join을 호출하는 형태이다.
* 스트림을 두 개로 나누어 연산할 경우 첫번째 스트림으로 CompletableFuture로 요청을 모두 보내놓고, 두번째 스트림은 이 Future의 결과를 하나씩 가져와 조합한다.

```java
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures = 
    shops.stream()
      .map(shop -> CompletableFuture.supplyAsync(
        () -> shop.getName() + "price is " + shop.getPrice(product)))
      .collect(toList());
      
  return priceFutures.stream()
    .map(CompletableFuture::join) //모든 비동기 동작이 끝나기를 기다린다.
    .collect(toList());
}
```

* CompletableFuture를 사용하더라도 병렬 스트림보다 성능이 느릴 수 있다.

### 커스텀 Executor

* 비동기 동작을 많이 사용하는 상황에서 유용하다.
* CompletableFuture는 병렬 작업에 이용할 수 있는 다양한 Executor를 직접 커스텀할 수 있어 스레드 풀의 크기를 조정하는 등 애플리케이션을 최적화할 수 있다.
* 스레드 풀 크기 조정
  * 스레드 풀의 크기가 너무 크면 context switching과 race condition으로 인해 CPU와 메모리 자원을 낭비할 수 있다.
  * 스레드 풀의 크기가 너무 작으면 CPU의 일부 코어가 활용되지 않을 수 있어 효율적이지 않다.
  *   자바 병렬 프로그래밍 책에 따르면 스레드 풀의 크기는 다음 공식으로 대략적인 CPU 활용 비율을 계산 할 수 있다.

      ```
      (스레드 개수) = (코어 수) * (CPU 활용 비율) * (1 + (대기시간)/(계산시간))
      - 코어 수는 Runtime.getRuntime().availableProcessors()를 통해 구할 수 있다.
      - CPU 활용 비율은 0과 1 사이의 값을 갖는다.
      ```
* 상점의 상품 가격 구하는 예제에서는 요청의 응답을 오래 기다리는 경우이므로 (대기시간)/(계산시간)이 100이라고 볼 수 있다.
* 스레드의 개수를 상점 수만큼만 두도록 한다. 스레드 수가 너무 많으면 서버에 충돌날 수 있기 때문에 하나의 Executor에서 사용할 스레드의 최대 개수는 100이하로 설정하도록 한다.

```java
private final Executor executor = Executors.newFixedThreadPool(Math.min(shops.size(), 100),
    new ThreadFactory() {
  public Thread new Thread(Runnable r) {
    Thread t = new Thread(r);
    t.setDeamon(true);
    return t;
  }
});
```

* 여기서 만든 스레드 풀에는 자바 프로그램이 종료될 때 강제로 종료되는 **데몬 스레드**를 포함한다.
* Executor를 정의하였으므로 CompletableFuture의 supplyAsync메서드에 이를 넣어주면, 4개 뿐만 아니라 400개의 상점을 탐색하는 경우에도 성능을 유지할 수 있다.

```java
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures = 
    shops.stream()
      .map(shop -> CompletableFuture.supplyAsync(
        () -> shop.getName() + "price is " + shop.getPrice(product), executor))
      .collect(toList());
      
  return priceFutures.stream()
    .map(CompletableFuture::join) //모든 비동기 동작이 끝나기를 기다린다.
    .collect(toList());
}
```

* 스트림 병렬화 vs CompletableFuture 병렬화
  * I/O가 포함되지 않은 계산 중심의 로직이면 스트림 인터페이스가 구현하기 간단하고 효율적
  * I/O를 기다리는 작업을 병렬로 수행할 때에는 CompletableFuture가 더 많은 유연성을 제공하며 대기시간/계산시간 비율에 적합한 스레드 수를 설정할 수 있다. 또한 스트림은 lazy 특성으로 인해 I/O를 실제 언제 처리할 지 예측하기 어렵다는 문제도 있다.

## 🏝️ 비동기 작업 파이프라인

### 할인 서비스 적용

* 앞서 다룬 예제의 상점들이 멤버 등급에 따라 서로 다른 할인율을 반환하는 할인 서비스를 사용하게 되었다.
* 아래는 할인 서비스를 사용하는 간단한 findPrices 메서드이다.
  * Quote라는 클래스는 어떤 가게의 어떤 가격에 어떤 할인율을 적용할지 담고 있다.
  * 이 정보들을 Discount 클래스로 넘기면 실제 할인이 적용된 가격을 조회할 수 있다. 이 과정에서 5초를 지연시킨다.
  * 이렇게 여러 가게로부터 가격을 가져와 할인을 적용한 값들을 리스트 형태로 반환한다.

```java
public List<String> findPrices(String product) {
  return shops.stream()
    .map(shop -> shop.getPrice(product)) // 각 상점에서 상품 가격 조회
    .map(Quote::parse) // 상점과 상품 가격, 할인규칙을 담은 문자열을 Quote 객체로 변환
    .map(Discount::applyDiscount)  // 상점 이름과 할인 적용된 상품 가격을 합친 문자열 반환
    .collect(toList());
}
```

```java
@Getter
@RequiredArgsConstructor
public class Quote {
  private final String shopName;
  private final double price;
  private final Discount.code discountCode;

  public static Quote parse(String s) {
    String[] split = s.split(":");
    String shopName = split[0];
    double price = Doule.parseDouble(split[1]);
    Discount.Code discountCode = Discount.Code.valueOf(split[2]);
    return new Quote(shopName, price, discountCode);
  }
}

public class Discount {
  public enum Code {
    NONE(0), SILVER(5), GOLD(10), PLATINUM(15), DIAMOND(20);
  }
  // ...
  
  public static String applyDiscount(Quote quote) {
    return quote.getShopName() + " price is " + Discount.apply(
      quote.getPrice(), quote.getDiscountCode());
  }
  
  private static double apply(double price, Code code) {
    delay(); // 1초 블록
    return format(price * (100 - code.percentage) / 100);
  }
}
```

* 이 방식은 순차적으로 한 상점에 가격 정보를 요청하고 할인 서비스를 적용하므로 가격 정보 요청과 할인 서비스에 각각 5초씩 걸린다면, 상점이 많아질수록 응답 시간이 기하급수적으로 늘어난다.
* 병렬 스트림을 사용하더라도 스레드 풀의 크기가 고정되어 있어 상점 수가 늘어났을 때 유연하게 대응할 수 없다.

### 동기 작업과 비동기 작업 조합

* 첫번째 map에서는 비동기적으로 가게에서 가격 정보를 조회한다. 이 때 CompletableFuture에는 예전에 정의했던 executor를 설정하여 가게 수만큼 스레드풀을 만들도록 한다.
* 두번째 map에서는 첫번째 map의 결과인 CompletableFuture\<String>에 thenApply() 메서드를 사용해 반환 값을 Completable\<Quote>로 변환한다. 이 때 thenApply() 메서드는 블록되지 않는다.
* 세번째 map에서는 원격 할인 서비스를 통해 할인된 가격을 얻어와야 하므로 동기적으로 작업을 수행해야 한다. thenCompose 메서드를 통해 quote 객체를 할인 서비스에 보내 할인된 가격을 담는 CompletableFuture를 반환한다.
* 이렇게 얻은 CompletableFuture가 완료되기를 기다렸다가 join으로 값을 추출하면 최종 값을 얻을 수 있다.

```java
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures = 
    shops.stream()
      .map(shop -> CompletableFuture.supplyAsync(
        () -> shop.getPrice(product), executor))
      .map(future -> future.thenApply(Quote::parse))
      .map(future -> future.thenCompose(quote ->
        CompletableFuture.supplyAsync(
          () -> Discount.applyDiscount(quote), executor)))
      .collect(toList());
      
  return priceFutures.stream()
    .map(CompletableFuture::join)
    .collect(toList());
}
```

* thenCompose 메서드도 Async 형태로 반환할 수 있지만 위 예제의 thenCompose의 CompletableFuture는 이전 CompletableFuture에 의존하기 때문에 Async버전으로 수행해도 실행 시간에는 영향이 없다. 따라서 스레드 전환 오버헤드가 적고 효율이 좋은 thenCompose를 사용했다.

### 독립적인 CompletableFuture들 합치기

* thenCombine
  * 두 개의 CompletableFuture 결과를 어떻게 합칠지에 대한 BiFunction을 두번째 인수로 받는 메서드
  * Async 버전도 별도로 존재하여, BiFunction이 정의하는 조합 동작이 스레드 풀으로 제출되면 별도 태스크에서 비동기적으로 수행된다.
* 아래 예시는 유로로 된 상품 정보를 얻어와 달러로 변환해야 하는 요구사항을 구현한 코드이다.
* 상품의 가격을 얻는 것을 하나의 비동기 작업으로 요청하고, thenCombine 메서드에 환율 정보를 가져오는 비동기 요청과 함께 어떻게 결과를 합칠지에 대한 함수를 입력받는다.
* 이로써 두 CompletableFuture의 결과가 생성되면 합칠 수 있는 코드가 되었다.

```java
Funtion<Double> futurePriceInUSD = CompletableFuture
  .supplyAsync(() -> shop.getPrice(product)) // 가격정보 요청
  .thenCombine(CompletableFuture.supplyAsync(
      () -> exchangeService.getRate(Money.EUR, Money.USD)), // 환율정보 요청
    (price, rate) -> price * rate)); //두 결과 합침
```

### Future와 CompletableFuture 비교

* CompletableFuture을 사용하면 람다 표현식을 사용해 동기/비동기 태스크를 활용한 복잡한 연산 수행 방법을 효과적으로 정의할 수 있다.
* 바로 위에서 다룬 유로로 된 상품 정보를 얻어와 달러로 변환하는 요구사항을 CompletableFuture와 람다 없이 구현하려면 아래와 같이 작성해야 하므로 코드의 복잡성이 높아진다.

```java
ExecutorService executor = Executors.newCachedThreadPool();
final Future<Double> futureRate = executor.submit(new Callable<Double>() {
  public Double call() {
    return exchangeService.getRate(Money.EUR, Money.USD);
  }
});

Future<Double> futurePriceInUSD = executor.submit(new Callable<Double>() {
  public Double call() {
    double priceInEUR = shop.getPrice(product);
    return priceInEUR * futureRate.get();
  }
});
```

### 타임아웃 처리

#### orTimeout() 메서드

* Future가 특정 시간 안에 작업을 끝내지 못할 경우 TimeoutException이 발생하도록 할 수 있다.
* 이를 통해 무한 블로킹되지 않도록 방지할 수 있다.
* CompletableFuture에서 제공하는 orTimeout 메서드는 지정된 시간 후 CompletableFuture를 TimeoutException으로 완료하고 또다른 CompletableFuture를 반환할 수 있도록 SchedluedThreadExecutor를 사용한다.

```java
Funtion<Double> futurePriceInUSD = CompletableFuture
  .supplyAsync(() -> shop.getPrice(product))
  .thenCombine(CompletableFuture.supplyAsync(
      () -> exchangeService.getRate(Money.EUR, Money.USD)),
    (price, rate) -> price * rate))
  .orTimeout(3, TimeUnit.SECONDS);
```

#### **completeOntimeout() 메서드**

* completeOntimeout 메서드를 사용해 환율 서비스가 1초안에 응답하지 않을 경우 기본 환율을 사용하도록 할 수 있다.
* 이를 통해 환율 서비스가 느릴 때마다 예외를 발생시키지 않고 정상적인 결과를 반환해줄 수 있다.

```java
Funtion<Double> futurePriceInUSD = CompletableFuture
  .supplyAsync(() -> shop.getPrice(product))
  .thenCombine(CompletableFuture
      .supplyAsync(() -> exchangeService.getRate(Money.EUR, Money.USD))
      .completeOnTimeout(DEFAULT_RATE, 1, TimeUnit.SECONDS),
    (price, rate) -> price * rate))
  .orTimeout(3, TimeUnit.SECONDS);
```

## 🏝️ CompletableFuture 종료 대응 방법

* 현실적으로 상품 가격 조회에 걸리는 시간은 각 가게마다 다를 수 있다.
* 모든 가게에서 가격 정보를 제공할 때까지 기다리지 말고 상점에서 가격을 제공할 때마다 즉시 보여주도록 구현한다.
* 기존에 구현했던 경우와 다르게 CompletableFuture을 join 처리 하지 않고 스트림을 그대로 반환한다.

```java
public Stream<CompletableFuture<String>> findPriceStream(String product) {
  return shop.stream()
    .map(shop -> CompletableFuture.supplyAsync(
      () -> shop.getPrice(product), executor))
    .map(future -> future.thenApply(Quote::parse))
    .map(future -> future.thenCompose(quote ->
      CompletableFuture.supplyAsync(
        () -> Discount.applyDiscount(quote), executor)));
}
```

* thenAccept 메서드는 CompletableFuture의 계산이 끝나면 값을 소비하도록 하는데, 연산 결과를 소비하는 Consumer를 인수를 받는데, 여기서는 값을 출력하도록 하였다.
* thenAccept 메서드의 반환 타입은 CompletableFuture\<Void>이다.

```java
findPriceStream("product1").map(f -> f.thenAccept(System.out::println));
```

* 상품 가격 조회가 가장 빨랐던 가게부터 가장 오래 걸렸던 가게의 상품 할인 가격을 모두 출력하려면 아래와 같이 `allOf` 메서드가 반환하는 CompletableFuture에 join을 호출하면 된다.
* 이를 통해 모든 상점의 결과가 반환되거나, 타임아웃으로 예외가 발생된 사실을 알릴 수 있다.

```java
CompletableFuture[] futures = findPriceStream("product1")
  .map(f -> f.thenAccept(System.out::println))
  .toArray(size -> new CompletableFuture[size]);
CompletableFuture.allOf(futures).join();
```

* 반대로 여러 CompletableFuture 중 하나만 작업이 끝나면 되는 상황이면 `anyOf` 메서드가 반환하는 CompletableFuture에 join을 호출하여 값을 가져올 수 있다.
