# Lettuce 단일 서버, 클러스터 서버, 풀링 사용 방법

### 들어가며

Lettuce란 Redis 캐시 서버를 사용하기 위한 Java Client 구현체로 네트워크 통신은 Netty를 기반으로 하고 비동기와 리액티브 API를 잘 제공해주는 라이브러리이다.&#x20;

가장 기본이 되는 단일 서버에 대한 연결과 클러스터 서버에 대한 연결을 지원한다.

나아가 Pooling을 통해 다수의 연결을 미리 맺어두고 사용할 수 있는 기능도 제공하고 있다.

### 단일(standalone) 서버 사용하기

* 가장 간단한 단일 서버 형태로 Redis를 구동해 사용하는 경우, Redis URI를 String 형태로 넘겨 RedisClient 객체를 생성한다.
* RedisClient로부터 연결 객체를 얻고 실제로 Redis 연산을 요청할 Commands 객체를 얻는다.
* Commands 객체에서는 다양한 API를 제공하기 때문에 원하는 Redis 연산을 요청할 수 있다.

```java
RedisClient redisClient = RedisClient.create("redis://password@localhost:6379/0");
StatefulRedisConnection<String, String> connection = redisClient.connect();
RedisCommands<String, String> syncCommands = connection.sync();

syncCommands.set("key", "Hello, Redis!");

connection.close();
redisClient.shutdown();
```

### 클러스터 서버 사용하기

* 단일 서버와 비슷하게, Redis URI를 넘겨 RedisClusterClient 객체를 생성한다.
* RedisClient로부터 연결 객체를 얻고 실제로 Redis 연산을 요청할 Commands 객체를 얻는다.
* Commands 객체에서는 다양한 API를 제공하기 때문에 원하는 Redis 연산을 요청할 수 있다.

```java
RedisURI redisUri = RedisURI.Builder.redis("localhost")
  .withPassword("authentication").build();
RedisClusterClient clusterClient = RedisClusterClient.create(rediUri);
StatefulRedisClusterConnection<String, String> connection = clusterClient.connect();
RedisAdvancedClusterCommands<String, String> syncCommands = connection.sync();

connection.close();
clusterClient.shutdown();
```

### 풀링 사용하기

* 아래와 같이 Redis 서버와의 연결을 풀링하여 사용할 수 있다.
* connection을 반환하는 supplier를 인자로 주어 connection pool에 원하는 개수 만큼 connection을 추가할 수 있도록 한다.
* Redis에 요청을 보낼 때에는 Pool으로부터 connection을 하나 받아와 해당 커넥션 내부의 Commands 객체를 사용해 API를 호출하도록 한다.

```java
RedisClient client = RedisClient.create();

CompletionStage<AsyncPool<StatefulRedisConnection<String, String>>> poolFuture
	= AsyncConnectionPoolSupport.createBoundedObjectPoolAsync(
		() -> client.connectAsync(StringCodec.UTF8, RedisURI.create(host, port)), BoundedPoolConfig.create());
		
CompletableFuture<TransactionResult> transactionResult = pool.acquire().thenCompose(connection -> {
		RedisAsyncCommands<String, String> async = connection.async();
		
		async.multi();
		async.set("key", "value");
		async.set("key2", "value2");
		return async.exec().whenComplete((s, throwable) -> pool.release(connection));
});

// terminating
pool.closeAsync();

// after pool completion
client.shutdownAsync();
```

* Redis 서버는 싱글스레드로 동작하기 때문에 대부분의 케이스에서 커넥션 풀을 두더라도 애플리케이션 성능에 크게 영향을 미치지 않는다. 다만 트랜잭션 기능을 사용하는 경우 처리하는 스레드 개수가 동적일 수 있으므로 이 때에는 동적 커넥션 풀을 사용해야 한다.
