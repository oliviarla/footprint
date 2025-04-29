# Consumer

## 개념

* 프로듀서가 브로커에 전송한 데이터들을 가져와 사용하기 위해 브로커에 요청을 보내는 클라이언트이다.
* 1개 이상의 컨슈머로 이뤄진 컨슈머 그룹을 운영하거나 토픽의 특정 파티션만 구독하는 컨슈머를 운영할 수 있다.

### 컨슈머 그룹

* 컨슈머 그룹으로 묶인 컨슈머들은 토픽의 여러 파티션들에 할당되어 데이터를 가져갈 수 있다. 그리고 1개의 파티션은 최대 1개의 컨슈머에만 할당이 가능하다.
* 따라서 컨슈머 그룹의 컨슈머 개수는 토픽의 파티션 개수보다 같거나 작아야 한다. 예를 들어 3개의 파티션을 갖는 토픽을 효과적으로 처리하려면 3개 이하로 이뤄진 컨슈머 그룹으로 운영해야 한다.
* 현재 운영하고 있는 토픽의 데이터가 어디에 적재되는지, 어떻게 처리되는지 파악하여 컨슈머 그룹을 최대한 나누는 것이 좋다.
  * 예를 들어 elastic search와 hadoop에 데이터를 동기로 적재하던 파이프라인을 카프카를 통해 각각의 저장소에 비동기로 적재하도록 변경할 수 있다. 이 때 컨슈머 그룹을 분리하지 않으면 한쪽 저장소에 장애가 발생했을 때 적재에 대해 지연이 발생할 수 있다.

### 리밸런싱

* **컨슈머가 추가/제거되는 상황에 리밸런싱이 발생**한다.
* 컨슈머 그룹의 컨슈머 중 일부에 장애가 발생(ex. 연결 불가)하면 해당 컨슈머에 할당되어 있던 파티션은 장애가 발생하지 않은 컨슈머에 소유권이 넘어간다.
* 언제든지 발생할 수 있기 때문에 데이터 처리 중 발생한 리밸런싱에 대응하는 코드를 컨슈머 클라이언트에 작성해두어야 한다.
* 파티션의 소유권을 재할당할 때 다른 컨슈머들이 토픽의 데이터를 읽을 수 없으므로 리밸런싱은 자주 발생하면 안된다.
* 카프카 브로커 중 한 프로세스가 **그룹 코디네이터**가 되어 컨슈머 그룹의 컨슈머 추가/삭제를 감지하고 리밸런싱을 발동시킨다.

### 커밋

* 컨슈머가 브로커로부터 어느 데이터까지 가져갔는지 **커밋**을 통해 기록한다.
* 카프카 브로커의 내부 토픽인 `__consumer_offsets` 에 특정 토픽의 파티션을 어떤 컨슈머 그룹이 어느 레코드까지 읽었는지에 대한 정보가 기록된다. 만약 이 토픽에 오프셋 커밋이 기록되지 않는다면 데이터를 중복해서 처리할 수 있기 때문에 **컨슈머 애플리케이션은 오프셋 커밋이 정상적으로 처리되었는지 검증해야 한다.**
*   비명시 오프셋 커밋

    * 일정 간격마다 오프셋을 커밋하도록 하는 것이다.
    * poll 메서드가 호출되었을 때 `auto.commit.interval.ms` 만큼 주기가 지났다면 해당 시점까지 읽은 레코드의 오프셋을 커밋한다.
    * 이 방식은 편리하지만 poll 메서드 호출 이후에 리밸런싱 또는 컨슈머 강제 종료 발생 시 컨슈머가 처리하는 데이터가 중복되거나 유실될 수 있다.

    <figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>
* 명시 오프셋 커밋
  * poll 메서드 호출 이후 반환받은 데이터의 처리가 완료되면 commitSync/commitAsync 메서드를 호출하여 가져온 레코드의 가장 마지막 오프셋을 기준으로 커밋을 수행하도록 한다.
  * commitSync의 경우 커밋 요청 및 응답에 시간이 오래 걸릴 수 있으며, commitAsync의 경우 커밋 요청이 실패했을 때 현재 처리 중인 데이터 순서를 보장하지 못해 데이터 중복 처리가 발생할 수 있다.

## Kafka Client로 컨슈머 구현하기

*   자바를 통해 컨슈머를 구현하기 위해서는 Kafka Client의 클래스들을 사용할 수 있다.

    * 컨슈머 그룹을 지정하여 컨슈머의 목적을 구분할 수 있다. 컨슈머 그룹을 기준으로 컨슈머 오프셋을 관리하기 때문에, subscribe 메서드를 통해 토픽을 구독하는 경우 컨슈머 그룹을 선언하여 프로그램이 재시작되더라도 해당 오프셋 이후의 데이터부터 처리되도록 해야 한다.
    * 프로듀서에서 지정한 직렬화 타입으로 역직렬화해야 한다.
    * subscribe 메서드는 컨슈머에게 토픽을 할당한다.
    * poll 메서드는 데이터를 브로커로부터 가져와 ConsumerRecord 타입을 반환한다. 이 때 Duration 타입을 인자로 받아 컨슈머 버퍼에 데이터를 기다리기 위한 타임아웃 간격을 설정한다.

    ```java
    Properties configs = new Properties();
    configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    configs.put(ConsumerConfig.GROUP_ID_CONFIG, "test-group");
    configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

    KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
    consumer.subscribe(Arrays.asList("testtopic"));

    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofSecond(1));
        for (ConsumerRecord<String, String> record : records) {
            System.out.println(record);
        }
    }
    ```
* poll 메서드를 호출하는 시점에 데이터를 브로커로부터 가져오는 것이 아니라 컨슈머 애플리케이션 실행 시 생성되는 Fetcher 인스턴스에 의해 미리 레코드들을 내부 큐에 저장해두고 사용자가 poll 메서드를 호출하면 내부 큐에 있는 레코드를 반환한다.

### 명시적 커밋

#### 동기 커밋

* 아래와 같이 동기 방식의 커밋 메서드를 호출할 수 있다.

```java
// ...
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSecond(1));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record);
    }
    consumer.commitSync();
}
```

* 개별 레코드 단위로 매번 오프셋을 커밋하고 싶다면 동기 방식의 커밋 메서드에 파티션객체와 오프셋 객체 맵을 입력하면 된다.
* 이 때 현재 처리한 오프셋에 1을 더한 값을 오프셋으로 입력해야 한다. 왜냐하면 컨슈머가 poll 메서드를 호출할 때 마지막 커밋 오프셋부터 읽기 시작하기 때문이다.

```java
// ...
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSecond(1));
    Map<TopicPartition, OffsetAndMetadata> currentOffset = new HashMap<>();
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record);
        currentOffset.put(
            new TopicPartition(record.topic(), record.partition()),
            new OffsetAndMetadata(record.offset() + 1, null));
        consumer.commitSync(currentOffset);
    }
}
```

#### 비동기 커밋

* 비동기 방식의 커밋 메서드에는 콜백 메서드를 가진 객체를 넣어 호출할 수 있다.

```java
// ...
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSecond(1));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record);
    }
    // consumer.commitAsync(); // 기본 호출
    consumer.commitAsync(new OffsetCommitCallback() {
        public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception e) {
            if (e != null) {
                logger.error("Commit failed for offsets {}", offsets, e);
            }
        }
    });
}
```

### 리밸런스 처리

* 리밸런스 발생 시 현재까지 처리된 데이터를 기준으로 커밋을 시도해야 한다.
* onPartitionAssigned 메서드는 리밸런스 완료 후 파티션이 할당되면 호출되는 메서드이고, onPartitionRevoked 메서드는 리밸런스가 시작되기 직전에 호출되는 메서드이다.

```java
public static void main(String[] args) {
    KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
    consumer.subscribe(Arrays.asList("testtopic"), new RebalanceListener());
    
    // ...
}
private static class RebalanceListener implements ConsumerRebalanceListener {
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        logger.warn("Partitions are assigned");
    }
    
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        logger.warn("Partitions are revoked");
        consume.commitAsync(currentOffsets);
    }
}
```

### 파티션 할당

* subscribe 메서드 대신 assign 메서드를 사용해 특정 파티션을 컨슈머에 직접 할당시킬 수 있다.

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
consumer.assign(Collections.singleton(new TopicPartition(TOPIC_NAME, PARTITION_NUMBER));
```

* 컨슈머에 할당된 파티션을 확인하려면 assignment 메서드를 사용하면 된다.

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
consumer.subscribe(Arrays.asList("testtopic"));
Set<TopicPartition> assignedPartitions = consumer.assignment();
```

### 안전한 종료

* 셧다운 훅을 구현해 wakeup 메서드를 호출하도록 하고, poll 메서드에서 WakeupException이 발생한 후에 close 메서드를 호출해 리소스가 정상적으로 종료되도록 한다.

```java
try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofSecond(1));
        for (ConsumerRecord<String, String> record : records) {
            System.out.println(record);
        }
    }
} catch (WakeupException) {
    // 리소스 종료 처리
} finally {
    consumer.close();
}
```

### 어드민 API 활용

* AdminClient 클래스를 통해 내부 옵션들을 설정하거나 조회할 수 있다.&#x20;
* 브로커 정보, 토픽 리스트, 컨슈머 그룹 등을 조회할 수 있고, 신규 토픽을 생성하거나 파티션 개수를 변경하거나 접근 제어 규칙(ACL) 등을 설정할 수 있다.
* 컨슈머나 프로듀서는 이 옵션을 사용해 상황에 맞게 적절한 처리를 할 수 있다.

```java
Properties configs = new Properties();
configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
AdminClient admin = AdminClient.create(configs);

for (Node node : admin.describeCluster().nodes().get()) {
    // 노드 정보를 통한 작업 수행
}

admin.close();
```

## 주요 옵션

### 필수 옵션

* bootstrap.servers
  * 프로듀서가 데이터를 전송한 카프카 브로커 클러스터의 주소
  * 2개 이상의 주소를 입력하여 일부 브로커에 이슈가 발생해도 접속에 문제가 없도록 하는 게 좋다.
* key.deserializer
  * 레코드의 메시지 키 역직렬화 클래스 지정
* value.deserializer
  * 레코드의 메시지 값 역직렬화 클래스 지정

### 선택 옵션

* group.id
  * 컨슈머 그룹 아이디
  * subsecribe() 메서드로 토픽을 구독할 경우 반드시 선언해주어야 한다.
  * 기본값은 null이다.
* auto.offse.reset
  * 저장된 오프셋이 없는 경우 어디서부터 읽을지 선택한다.
  * 다음과 같은 타입이 있다.
    * latest : 가장 최근에 넣은 높은 번호의 오프셋부터 읽기
    * earliest: 가장 오래전에 넣은 낮은 번호의 오프셋부터 읽기
    * none : 컨슈머 그룹이 커밋한 기록을 확인해 없는 경우 에러를 반환하고, 있으면 기존 커밋 기록 이후의 오프셋부터 읽기
  * 기본값은 latest이다.
* enable.auto.commit
  * 자동 커밋 동작 여부를 설정한다.
  * 기본값은 true이다.
* auto.commit.interval.ms
  * 자동 커밋일 경우의 커밋 간격을 설정한다.
  * 기본값은 5000이다.
* max.poll.records
  * poll() 호출 시 반환되는 레코드 개수를 지정한다.
  * 기본값은 500이다.
* session.timout.ms
  * 컨슈머와 브로커의 연결이 끊겨도 유지하는 최대시간을 지정한다.
  * 이 시간 내에 하트비트를 브로커에 보내지 않으면 브로커는 컨슈머에 이상이 생겼다고 간주하고 리밸런싱을 시작한다.
  * 보통 하트비트 시간 간격의 3배로 지정한다.
  * 기본값은 10000ms이다.
* heartbeat.interval.ms
  * 하트비트를 전송하는 간격을 지정한다.
* max.poll.interval.ms
  * poll 메서드 호출 간격의 최대 시간을 지정한다.
  * poll 메서드 호출 이후 데이터 처리에 시간이 너무 많이 걸리는 경우 비정상으로 판단하여 리밸런싱을 시작한다.
  * 기본값은 300000ms(5분)이다.
* isolation.level
  * 트랜잭션 프로듀서가 레코드를 트랜잭션 단위로 보낼 경우 고립 수준을 지정한다.
  * read\_commited: 커밋 완료된 레코드만 읽기
  * read\_uncommited: 커밋 여부에 상관없이 레코드를 읽기
  * 기본값은 read\_commited이다.

## 멀티 스레드 컨슈머

* 1개의 파티션에 1개의 컨슈머가 할당되어 처리되므로, n개의 파팃견이 있다면 동일 컨슈머 그룹으로 묶인 n개의 스레드를 운영할 수 있다.
* 즉, n개 스레드를 가진 하나의 프로세스 혹은 1개 스레드를 가진 n개의 프로세스를 운영해야 한다.
* 하나의 프로세스에 여러 스레드를 가지도록 할 때 하나의 컨슈머 스레드에서 OOM같은 예외 상황이 발생하면 프로세스가 종료되므로 다른 스레드들도 종료된다.

### 멀티 워커 스레드 전략

* 1개의 컨슈머 스레드와 n개의 데이터 처리를 담당하는 워커 스레드를 실행하는 방법이다.
* 브로커로부터 전달받은 레코드들을 병렬로 처리하면 1개 컨슈머 스레드로 받은 데이터를 더 빠르게 처리할 수 있다.

#### 사용 방법

* Runnable 인터페이스를 구현한 ConsumerWorker 클래스는 컨슈머의 poll 메서드를 통해 받은 레코드들을 병렬로 처리할 수 있도록 한다.
  * newCachedThreadPool은 필요한 만큼 스레드 풀을 늘려 실행하는 방식으로, 짧은 시간의 생명 주기를 가진 스레드를 사용할 때 유용하다.

```java
public class ConsumerWorker implements Runnable {

    private final static Logger logger = LoggerFactory.getLogger(ConsumerWorker.class);
    private String recordValue;

    ConsumerWorker(String recordValue) {
        this.recordValue = recordValue;
    }

    @Override
    public void run() {
        // record를 처리하는 로직이 들어간다.
        logger.info("thread:{}\trecord:{}", Thread.currentThread().getName(), recordValue);
    }
}
```

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);
consumer.subscribe(Arrays.asList(TOPIC_NAME));
ExecutorService executorService = Executors.newCachedThreadPool();
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(10));
    for (ConsumerRecord<String, String> record : records) {
        ConsumerWorker worker = new ConsumerWorker(record.value());
        executorService.execute(worker);
    }
}
```

#### 한계

* 스레드들에서 데이터 처리가 끝나지 않더라도 다음 poll 메서드를 호출하므로 완전히 데이터 처리가 끝나지 않더라도 커밋하게 된다. 따라서 리밸런싱, 컨슈머 장애 발생 시 데이터가 유실될 수 있다.
* 레코드별로 스레드 생성은 순차로 진행되지만 각 스레드가 레코드를 처리 완료하는 순서는 변할 수 있어 레코드 처리의 역전 현상이 발생할 수 있다. 따라서 서버 리소스 모니터링이나 IoT서비스 센서 데이터 수집 파이프라인 등 데이터 역전 현상이 발생하더라도 빠른 처리 속도만 있으면 되는 경우에 사용해야 한다.

### 컨슈머 멀티 스레드 전략

* poll 메서드를 호출하는 스레드를 여러 개 띄우는 방법이다.
* 컨슈머 스레드를 늘려 운영하면 각 스레드에 파티션이 할당된다.
* 구독하려는 토픽의 파티션 개수만큼만 컨슈머 스레드를 운영해야 한다. 만약 컨슈머 스레드가 파티션 개수보다 많으면 파티션에 할당되지 못한 컨슈머 스레드는 데이터 처리를 하지 않게 된다.
* Runnable 인터페이스를 구현한 ConsumerWorker 클래스를 필요한 만큼 생성해 병렬로 컨슈머 스레드가 돌아가도록 한다.
  * KafkaConsumer 클래스는 thread-safe하지 않으므로 스레드별로 KafkaConsumer 클래스를 생성해 사용해야 한다.
  * newCachedThreadPool을 통해 스레드 풀을 구성해 내부 작업이 완료되면 스레드를 종료하도록 한다.

```java
public class ConsumerWorker implements Runnable {

    private final static Logger logger = LoggerFactory.getLogger(ConsumerWorker.class);

    private final Properties prop;
    private final String topic;
    private final String threadName;
    private KafkaConsumer<String, String> consumer;

    ConsumerWorker(Properties prop, String topic, int number) {
        this.prop = prop;
        this.topic = topic;
        this.threadName = "consumer-thread-" + number;
    }

    @Override
    public void run() {
        consumer = new KafkaConsumer<>(prop);
        consumer.subscribe(Arrays.asList(topic));
        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
                for (ConsumerRecord<String, String> record : records) {
                    logger.info("{}", record);
                }
                consumer.commitSync();
            }
        } catch (WakeupException e) {
            System.out.println(threadName + " trigger WakeupException");
        } finally {
            consumer.commitSync();
            consumer.close();
        }
    }

    public void shutdown() {
        consumer.wakeup();
    }
}
```

```java
int CONSUMER_COUNT = 3;
Properties configs = new Properties();
configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
configs.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
configs.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

ExecutorService executorService = Executors.newCachedThreadPool();
for (int i = 0; i < CONSUMER_COUNT; i++) {
    ConsumerWorker worker = new ConsumerWorker(configs, TOPIC_NAME, i);
    workerThreads.add(worker);
    executorService.execute(worker);
}
```

## 컨슈머 랙

* 컨슈머 랙은 토픽의 최신 오프셋과 컨슈머 오프셋 간의 차이를 의미한다.
* 컨슈머가 정상 동작하는지 여부를 확인하는 지표 중 하나이다.
* 아래와 같이 프로듀서가 가장 최근에 추가한 레코드와 컨슈머가 가장 최근에 읽은 레코드 간의 차이를 LAG이라고 한다.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><a href="https://docs.redhat.com/en/documentation/red_hat_streams_for_apache_kafka/2.0/html/deploying_and_upgrading_amq_streams_on_openshift/assembly-metrics-setup-str">https://docs.redhat.com/en/documentation/red_hat_streams_for_apache_kafka/2.0/html/deploying_and_upgrading_amq_streams_on_openshift/assembly-metrics-setup-str</a></p></figcaption></figure>

* 컨슈머 그룹과 토픽, 파티션 별로 컨슈머 랙이 존재하게 된다.
* 프로듀서가 보내는 데이터 양이 컨슈머 데이터 처리량보다 많으면 컨슈머 랙은 늘어나게 된다. 반대로 프로듀서가 보내는 데이터 양이 컨슈머 데이터 처리량보다 적으면 컨슈머 랙은 줄어들게 된다.
* 컨슈머 랙이 0이면 지연이 없는 상태임을 의미한다.
* 컨슈머 랙을 모니터링하여 컨슈머의 장애를 확인하고 파티션 개수를 정하는 데에 참고할 수 있다.
* 만약 컨슈머 랙이 늘어난다면 지연을 줄이기 위해 일시적으로 파티션, 컨슈머 개수를 늘려 병렬 처리량을 늘려야 한다.

### 컨슈머 랙 확인 방법

#### 카프카 명령어 사용

* 아래와 같은 명령어를 사용해 특정 컨슈머 그룹의 상태를 확인할 수 있다.
* 이 방식으로는 지표를 지속적으로 기록하고 모니터링하기 어렵다.

```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-group --describe
```

#### 컨슈머 애플리케이션의 metrics 메서드 사용

* 컨슈머에 메트릭 정보 조회 요청을 보내기 위해 metrics 메서드를 호출하고 이를 로깅할 수 있다.
* 컨슈머 애플리케이션이 비정상적으로 종료된 상황이라면 컨슈머 랙을 더이상 모니터링할 수 없고, 컨슈머 애플리케이션을 분산해 운영할 경우 모든 애플리케이션에 호출 로직을 추가해주어야 하는 등의 단점이 있다.

```java
for (Map.Entry<MetricName, ? extends Metric> entry: kafkaConsumer.metrics().entrySet()) {    
    if ("records-lag-max".equals(entry.getKey().name()) |
        "records-lag".equals(entry.getKey().name()) |
        "records-lag-avg".equals(entry.getKey().name())) {
        Metric metric = entry.getValue();
        logger.info("{}:{}", entry.getKey().name(), metric.metricValue());
    }
}
```

#### 외부 모니터링 툴 사용

* 데이터독 등 카프카 클러스터 종합 모니터링 툴을 사용하면 카프카 운영에 필요한 다양한 지표를 모니터링할 수 있다.
* 외부 모니터링 툴을 이용하면 카프카 클러스터에 연결된 모든 컨슈머와 토픽들의 랙 정보를 한 번에 모니터링할 수 있다.
* 컨슈머 랙이 임계치에 도달할 때 마다 알람을 보내는 것은 무의미하다. 일시적으로 프로듀서가 데이터를 많이 보내면 임계치가 넘어갈 수 있지만 컨슈머 또는 파티션에 문제가 발생했다고 간주하기는 어렵기 때문이다.
* 컨슈머 랙과 파티션의 오프셋을 슬라이딩 윈도우 방식으로 계산하여 파티션과 컨슈머의 상태를 표현하면 문제가 발생한 부분을 감지할 수 있다.
* 링크드인이 제공하는 카프카 모니터링 툴인 버로우에서는 슬라이딩 윈도우 방식을 통해 최신 오프셋 증가량에 비해 컨슈머 오프셋 증가량이 미미하면 컨슈머를 WARNING 상태로 표현하고, 컨슈머 오프셋이 아예 증가하지 않는다면 파티션을 STALLED 상태로 표현하고 컨슈머는 ERROR 상태로 표현한다.

### 컨슈머 랙 모니터링 아키텍처

* 버로우, 텔레그래프, 엘라스틱서치, 그라파나를 이용해 컨슈머 랙 모니터링 아키텍처를 구성할 수 있다.
* 텔레그래프는 버로우를 통해 컨슈머 랙을 조회하고 엘라스틱서치에 저장한다. 그라파나는 엘라스틱서치의 데이터를 시각화하고 조건에 따라 알림을 보낼 수 있다.

## 컨슈머 배포 프로세스

* 컨슈머 애플리케이션 운영 시 로직 변경으로 인해 배포를 해야한다면 중단 배포, 무중단 배포 중 하나를 사용하면 된다.

### 중단 배포

* 컨슈머 애플리케이션을 완전히 종료 후 다시 배포하는 방식이다.
* 기존 컨슈머 애플리케이션이 종료된 후 신규 컨슈머 애플리케이션이 배포되기 전 사이에 컨슈머 랙이 늘어날 수 있으므로 파이프라인을 운영하는 서비스가 오래 중단되지 않도록 조심해야 한다.
* 신규 애플리케이션의 실행 전후를 명확히 특정 오프셋 지점으로 나눌 수 있다. 이를 통해 새로 배포한 애플리케이션에 이슈가 생겨도 롤백하고 데이터를 재처리할 수 있다.

### 무중단 배포

* 컨슈머가 중단되면 안되는 애플리케이션에서 유용하며, 배포를 위해 신규 서버를 발급받아야 한다.
* 블루/그린 배포
  * 기존 컨슈머 애플리케이션을 종료하지 않고 신규 컨슈머 애플리케이션을 실행시킨 후 트래픽을 전환하는 방식이다.
  * 파티션 개수와 컨슈머 개수를 동일하게 운영하고 있다면, 신규 컨슈머 애플리케이션을 배포하고 동일 컨슈머그룹으로 파티션을 구독하도록 하여 idle 상태로 두고, 모든 준비가 완료되면 기존 애플리케이션을 중단하여 리밸런싱되도록 하면 된다.
  * 리밸런스가 한 번만 발생하므로 많은 수의 파티션을 운영하는 경우에도 빠르게 배포를 수행할 수 있다.
* 롤링 배포
  * 신규 컨슈머 하나를 실행하고 기존 컨슈머 하나를 종료시키는 작업을 반복해서 실행하고 모니터링하여 적은 장비로도 효과적으로 무중단 배포를 할 수 있는 방법이다.
  * 파티션 개수가 컨슈머 애플리케이션을 띄우는 장비 개수보다 같거나 커야한다.
  * 파티션 개수가 많아질수록 리밸런스 시간이 길어진다. 따라서 파티션 개수가 많지 않은 경우에 유용하다.
* 카나리 배포
  * 여러 파티션 중 일부 파티션에 신규 컨슈머를 배정해 일부 데이터만 신규 컨슈머가 처리하도록 하고, 문제가 없다면 전체적으로 신규 컨슈머를 블루그린 혹은 롤링 방식으로 배포하는 방법이다.

## 스프링 카프카 컨슈머

* 스프링 카프카 컨슈머에는 2개의 타입과 7개의 커밋 종류가 있다.
* 레코드 리스너
  * 1개의 레코드를 처리한다.
* 배치 리스너
  * 카프카 클라이언트에서 제공하는 poll 메서드처럼 한 번에 여러 레코드를 처리한다.
* 이외에도 각 리스너로부터 파생된 형태들이 존재한다. 매뉴얼 커밋을 사용할 경우 Acknowledging이 붙은 리스너를 사용하고, KafkaConsumer 객체에 직접 접근해 사용할 경우 ConsumerAware가 붙은 리스너를 사용하면 된다.
  * 레코드 리스너 파생
    * AcknowledgingMessageListener
    * ConsumerAwareMessageListener
    * AcknowledgingConsumerAwareMessageListener
  * 배치 리스너 파생
    * BatchAcknowledgingMessageListener
    * BatchConsumerAwareMessageListener
    * BatchAcknowledgingConsumerAwareMessageListener
* 커밋의 종류는 다음과 같으며, 스프링 카프카에서는 AckMode라는 속성으로 원하는 커밋 방식을 지정할 수 있다.
  * RECORD
    * 레코드 단위로 프로세싱 후 커밋
  * BATCH (기본값)
    * poll 메서드로 호출된 레코드가 모두 처리된 후 커밋
  * TIME
    * 특정 시간 이후 커밋
    * AckTime 옵션을 같이 지정해야 한다.
  * COUNT
    * 특정 개수만큼 레코드가 처리된 후 커밋
    * AckCount 옵션을 같이 지정해야 한다.
  * COUNT\_TIME
    * TIME, COUNT 중 하나라도 조건이 맞으면 커밋
  * MANUAL
    * Acknowldegement.acknowledge 메서드가 호출되면 다음 poll 메서드 호출 시 커밋한다.
    * 리스너를 AcknowledgingMessageListener 혹은 BatchAcknowledgingMessageListener로 두어야 한다.
  * MANUAL\_IMMEDIATE
    * Acknowldegement.acknowledge 메서드가 호출되면 즉시 커밋한다.
    * 리스너를 AcknowledgingMessageListener 혹은 BatchAcknowledgingMessageListener로 두어야 한다.

### 리스너 생성하기

* 리스너를 사용하려면, 기본 리스너 컨테이너를 사용하거나 컨테이너 팩토리로 직접 리스너 컨테이너를 생성해 사용해야 한다.
* 다음은 **레코드 리스너**를 활용한 예제이다.
  * application.properties에 spring.kafka.listener.type을 RECORD로 지정해주어야 한다.
  * topics, groupId를 지정하여 레코드를 리스너 내부에서 처리할 수 있다.

```java
// 레코드를 파라미터로 받는 가장 기본적인 리스너
@KafkaListener(topics = "test",
    groupId = "test-group-00")
public void recordListener(ConsumerRecord<String,String> record) {
    logger.info(record.toString());
}

// 메시지 값을 파라미터로 받는 리스너
@KafkaListener(topics = "test",
    groupId = "test-group-01")
public void singleTopicListener(String messageValue) {
    logger.info(messageValue);
}

// properties 속성으로 카프카 컨슈머 옵션 값을 부여 가능
@KafkaListener(topics = "test",
    groupId = "test-group-02", properties = {
    "max.poll.interval.ms:60000",
    "auto.offset.reset:earliest"
})
public void singleTopicWithPropertiesListener(String messageValue) {
    logger.info(messageValue);
}

// concurrency 속성으로 지정한 스레드 수 만큼 컨슈머 스레드를 만들어 병렬 처리 지원
@KafkaListener(topics = "test",
    groupId = "test-group-03",
    concurrency = "3")
public void concurrentTopicListener(String messageValue) {
    logger.info(messageValue);
}

// 특정 토픽의 특정 파티션, 특정 파티션의 오프셋을 지정해 구독 가능
@KafkaListener(topicPartitions = {
            @TopicPartition(topic = "test01", partitions = {"0", "1"}),
            @TopicPartition(topic = "test02", partitionOffsets = @PartitionOffset(partition = "0", initialOffset = "3"))
    },
    groupId = "test-group-04")
public void listenSpecificPartition(ConsumerRecord<String, String> record) {
    logger.info(record.toString());
}
```

* 다음은 **배치 리스너**를 활용한 예제이다.
  * application.properties에 spring.kafka.listener.type을 BATCH로 지정해주어야 한다.
  * 레코드 리스너와 달리 레코드를 List 혹은 ConsumerRecords를 통해 가져온다.

```java
@KafkaListener(topics = "test",
        groupId = "test-group-01")
public void batchListener(ConsumerRecords<String, String> records) {
    records.forEach(record -> logger.info(record.toString()));
}

@KafkaListener(topics = "test",
        groupId = "test-group-02")
public void batchListener(List<String> list) {
    list.forEach(recordValue -> logger.info(recordValue));
}

@KafkaListener(topics = "test",
        groupId = "test-group-03",
        concurrency = "3")
public void concurrentBatchListener(ConsumerRecords<String, String> records) {
    records.forEach(record -> logger.info(record.toString()));
}
```

* 다음은 **BatchAcknowledgingConsumerAwareMessageListener** 사용 예제이다.
  * application.properties에 spring.kafka.listener.type을 BATCH로 지정한다.
  * ack-mode를 기본으로 둘 경우 리스너가 자동으로 커밋을 하기 때문에, 직접 커밋하는 코드를 두기 위해 spring.kafka.listener.ack-mode를 MANUAL\_IMMEDIATE로 지정한다.

```java
// 커밋을 수행하기 위해 Acknowledgement 객체를 인자로 받아 사용한다.
@KafkaListener(topics = "test", groupId = "test-group-01")
public void commitListener(ConsumerRecords<String, String> records, Acknowledgment ack) {
    records.forEach(record -> logger.info(record.toString()));
    ack.acknowledge();
}

// 동기 커밋, 비동기 커밋을 비롯한 컨슈머 메서드를 사용하기 위해 Consumer 객체를 인자로 받아 사용한다.
@KafkaListener(topics = "test", groupId = "test-group-02")
public void consumerCommitListener(ConsumerRecords<String, String> records, Consumer<String, String> consumer) {
    records.forEach(record -> logger.info(record.toString()));
    consumer.commitSync();
}
```

### 커스텀 리스너 컨테이너

* 서로 다른 설정을 가진 2개 이상의 리스너를 구현하거나 리밸런스 리스너를 구현하기 위해서는 커스텀 리스너 컨테이너를 구현해야 한다.
* KafkaListenerContainerFactory 객체를 생성해 빈으로 등록해주어야 한다.

```java
@Configuration
public class ListenerContainerConfiguration {

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> customContainerFactory() {

        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "my-kafka:9092");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        // 리스너 컨테이너 팩토리 생성 시 컨슈머 기본 옵션을 설정하기 위한 객체
        DefaultKafkaConsumerFactory cf = new DefaultKafkaConsumerFactory<>(props);
        
        // 리스너 컨테이너를 만들기 위해 직접적으로 사용되는 객체 / 이를 통해 2개 이상의 컨슈머 리스너를 만들 수 있다
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        
        // 리밸런스 리스너 선언을 위해 setConsumerRebalanceListener 메서드 호출
        factory.getContainerProperties().setConsumerRebalanceListener(new ConsumerAwareRebalanceListener() {
            @Override
            public void onPartitionsRevokedBeforeCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions) {

            }

            @Override
            public void onPartitionsRevokedAfterCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions) {

            }

            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {

            }

            @Override
            public void onPartitionsLost(Collection<TopicPartition> partitions) {

            }
        });
        factory.setBatchListener(false);
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.RECORD);
        factory.setConsumerFactory(cf);
        return factory;
    }
}
```

* 리스너 메서드에 빈으로 등록한 containerFactory 이름을 지정해준다.

```java
@KafkaListener(topics = "test",
    groupId = "test-group",
    containerFactory = "customContainerFactory")
public void customListener(String data) {
    logger.info(data);
}
```

**출처**

* 아파치 카프카 애플리케이션 프로그래밍
* [https://medium.com/apache-kafka-from-zero-to-hero/apache-kafka-guide-36-consumer-offset-commit-strategies-41ef6bf34fcd](https://medium.com/apache-kafka-from-zero-to-hero/apache-kafka-guide-36-consumer-offset-commit-strategies-41ef6bf34fcd)
