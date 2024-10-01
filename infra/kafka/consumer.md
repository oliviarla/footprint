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

    <figure><img src="../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>
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

**출처**

아파치 카프카 애플리케이션 프로그래밍

[https://medium.com/apache-kafka-from-zero-to-hero/apache-kafka-guide-36-consumer-offset-commit-strategies-41ef6bf34fcd](https://medium.com/apache-kafka-from-zero-to-hero/apache-kafka-guide-36-consumer-offset-commit-strategies-41ef6bf34fcd)
