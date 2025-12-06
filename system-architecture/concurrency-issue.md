# ⭐️. 재고 시스템으로 알아보는 동시성이슈 해결방법

## 동시성 문제

* 아래와 같이 1번 상품에 대해 동시에 100명이 주문하는 상황이 있다고 가정한다.
* 이 때 Stock 테이블에 존재하는 quantity 컬럼 값을 여러 스레드에서 동시에 접근 시도한다.
* 하지만 quantity 값에 무분별하게 접근하게 되어 race condition이 발생하게 된다.
* 아래 표를 보면, `Thread-1`이 quantity에 접근하여 수량을 1 낮추고자 한다. 하지만 이 때 `Thread-2`도 quantity에 접근하여 수량을 1 낮추려고 하다보니, 결국 quantity 값이 3이 아닌 4가 되어버리는 현상이 발생하는 것이다.

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

* 이 문제를 테스트해보려면 아래 코드를 동작시키면 된다.

```java
@Test
public void 동시에_100명이_주문() throws InterruptedException {
    int threadCount = 100;
    ExecutorService executorService = Executors.newFixedThreadPool(32);
    CountDownLatch latch = new CountDownLatch(threadCount);

    for (int i = 0; i < threadCount; i++) {
        executorService.submit(() -> {
            try {
                stockService.decrease(1L, 1L);
            } finally {
                latch.countDown();
            }
        });
    }

    latch.await();

    Stock stock = stockRepository.findById(1L).orElseThrow();

    // 100 - (100 * 1) = 0
    assertEquals(0, stock.getQuantity());
}
```

* ExcutorService: 비동기로 실행하는 작업을 단순화하여 사용할 수 있는 API
* CountDownLatch: 다른 스레드에서 진행중인 작업이 완료될 때까지 기다릴 수 있도록 한다.
* race condition: 여러 스레드가 동시에 공유 데이터에 접근해 변경하려할 때 발생하는 문제

## Synchronized 문제

* decrease 메소드 단에 synchronized 키워드를 붙여주면 해당 메소드는 한 번에 한개의 스레드만 접근 가능하게 된다.

```java
public class StockService {
	// ...
	public synchronized void decrease(final Long id, final Long quantity) {
		Stock stock = stockRepository.findById(id).orElseThrow();
		stock.decrease(quantity);
		stockRepository.saveAndFlush(stock);
	}
}
```

* 하지만 Transactional 어노테이션을 사용할 경우, Spring AOP에 의해 트랜잭션 내용이 추가된 래핑 클래스를 새로 만들어 아래와 같이 동작하게 된다.

```java
public class TransactionStockService {
	private StockService stockService;
	
	...
	
	public void decrease(Long id, Long quantity) {
		startTransaction();
		stockService.decrease(id, quantity);
		// 이 시점에 다른 스레드가 stockService.decrease()를 호출할 수 있다!
		endTransaction();
	}
}
```

* 주석과 같이 Transaction이 커밋되기 전인데 decrease 메서드가 수행완료된 시점이 존재하므로 동시성 문제가 완전히 해결되지 않는다.
* 각 프로세스 안에서만 보장하기 때문에 서버가 2대 이상일때 정합성을 보장해주지 못해 실무에서 거의 사용되지 않는다.

## Database Lock

### Pessimistic Lock

* 실제 데이터에 Lock을 걸어 정합성을 맞추는 방법
* exclusive lock을 걸면 다른 트랜잭션에서는 lock이 해제되기 전까지 절대 데이터를 읽을 수 없다.
* 데드락이 걸릴 수 있으므로 주의해야 한다.
* 충돌이 빈번하게 일어난다면 Optimistic Lock에 비해 성능이 좋을 수 있으나, 일반적인 경우 별도의 락을 잡기 때문에 성능 저하가 발생할 수 있다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Example> findById(Long id);
```

### Optimistic Lock

* 실제 Lock을 사용하지 않고 버전을 이용해 정합성을 맞추는 방법
* 데이터를 읽고 update를 수행할 때 데이터 읽은 시점의 버전과 현재 버전이 일치하는지 확인
* 데이터 읽은 시점 이후에 수정 사항이 생겨 버전이 변경되었다면 다시 최신 데이터를 읽은 후 작업을 수행해야 한다.
* 별도의 락을 잡지 않으므로 성능이 크게 저하되지 않으며, 충돌이 적게 일어난다면 Pessimistic Lock보다는 이 방법을 사용하는게 좋다.
* 실패했을 경우에 대해 어떻게 처리할 지 직접 코드를 작성해주어야 한다.

```java
public interface ExampleRepository extends JpaRepository<Example, Long> {
	@Lock(LockModeType.OPTIMISTIC)
	Optional<Example> findById(Long id);
}
```

```java
try {
	exampleRepository.findById(id);
} catch (Exception e) {
	Thread.sleep(50);
}
```

### Named Lock

* 이름을 가진 metadata locking
* 이름을 가진 lock을 획득한 후 해제할 때 까지 다른 세션은 lock을 획득할 수 없다
* transaction이 종료되더라도 lock이 자동 해제되지 않으므로, 직접 해제해주거나 선점 시간이 완료될 때까지 기다려야 한다.
* 분산 락 구현 시 자주 사용된다.
* 타임아웃을 구현하기 쉽다.
* 같은 데이터소스를 사용하면 커넥션 풀이 부족해져 다른 서비스에 영향이 갈 수 있으므로 분리해서 사용해야 한다.

```java
// 실무에서는 Named Lock 사용 시 별도의 JDBC Datasource를 준비해 사용할 것!
public interface ExampleRepository extends JpaRepository<Example, Long> {
    @Query(value = "select get_lock(:key, 3000)", nativeQuery = true)
    void getLock(String key);
    
    @Query(value = "select release_lock(:key)", nativeQuery = true)
    void releaseLock(String key);
}
```

```java
try {
  lockRepository.getLock(id.toString());
  Example example = exampleRepository.findById(id);
  example.updateName("new name!");
} finally {
  lockRepository.releaseLock(id.toString());
}
```

## Redis

* 동시성 문제를 해결하기 위해 in-memory cache인 레디스를 사용할 수 있다.
* 먼저 아래와 같이 레디스를 도커로 구동시킨다.

```bash
docker pull redis
docker run -name <내가 지정할 이름> -d -p 6379:6379 redis
docker ps // 이 명령어로 실행중인 redis 컨테이너 아이디를 얻을 수 있다.
docker exec -it <redis 컨테이너 아이디> redis-cli
```

* 그리고 Spring Project에 의존성을 추가한다.

```java
// Lettuce	
implementation("org.springframework.boot:spring-boot-starter-data-redis")

// Redisson
implementation("org.redisson:redisson-spring-boot-starter:3.23.2")
```

### Lettuce

* Lettuce는 Redis의 클라이언트 중 하나이다. Netty 기반으로 동작한다.
* spin lock 방식을 사용해 동시성을 제어할 것이며, mysql의 named lock 방식과 비슷하다.
* 구현이 간단하다는 장점이 있다.
* 먼저 lock과 unlock 메서드를 생성한다. Stock 엔티티의 PK를 key로 하고, "lock"이라는 문자열을 value로 하여 3초동안만 값이 존재하도록 한다.
* 만약 이미 key에 해당하는 value가 존재하면 lock이 잡혀있다는 뜻이다.

```java
@Component
public class RedisLockRepository {
  private RedisTemplate<String, String> redisTemplate;

  public Boolean lock(Long key) {
      return redisTemplate
              .opsForValue()
              .setIfAbsent(key.toString(), "lock", Duration.ofMillis(3_000));
  }
  public Boolean unlock(Long key) {
      return redisTemplate.delete(key.toString());
  }
}
```

* 이제 Stock 엔티티의 quantity를 감소시키기 위해서는 반드시 lock을 가진 스레드여야 한다.
* 100ms마다 lock 획득을 재시도하는 로직을 작성해주었다.

```java
public void decrease(Long key, Long quantity) throws InterruptedException {
  while (!redisLockRepository.lock(key)) {
      Thread.sleep(100); // lock 획득 여부를 확인하는 텀을 두어 Redis에 가는 부하를 줄여준다.
  }	
  // 락을 얻어 진입
  try {
    stockService.decrease(key, quantity);
  } finally {
    // 락 해제
    redisLockRepository.unlock(key);
  }
}
```

### Redisson

* pub-sub 기반으로 Lock 구현을 제공한다.
* pub-sub 채널을 통해 락을 점유중인 스레드가 락을 해제하면, 락을 획득하려는 스레드에게 알려준다.
* 락 획득 재시도를 기본으로 제공한다.
* 따라서 재시도가 필요한 경우에는 Redisson을, 재시도가 필요없다면 Lettuce를 사용하면 된다.

```java
$ subscribe ch1
Reading message ...

$ publish ch1 hello
```

* 10초동안 lock 점유 시도에 실패하면 decrease 메서드를 수행할 수 없다.
* lock을 점유했다면 decrease 메서드를 수행해 quantity를 감소시킨다.

```java
private RedissonClient redissonClient;

public void decrease(Long key, Long quantity) throws InterruptedException {
    RLock lock = redissonClient.getLock(key.toString());

    try {
      // 10초동안 시도하고, 1초동안 Lock 점유
      boolean available = lock.tryLock(10, 1, TimeUnit.SECONDS);
    
      if (!available) {
          // Lock 획득 실패
          return;
      }
    
      stockService.decrease(key, quantity);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    } finally {
    	// Lock 해제
      lock.unlock();
    }
}
```
