# Consumer

## 컨슈머

### 개념

* 프로듀서가 브로커에 전송한 데이터들을 가져와 사용하기 위해 브로커에 요청을 보내는 클라이언트이다.
* 1개 이상의 컨슈머로 이뤄진 컨슈머 그룹을 운영하거나 토픽의 특정 파티션만 구독하는 컨슈머를 운영할 수 있다.

#### 컨슈머 그룹

* 컨슈머 그룹으로 묶인 컨슈머들은 토픽의 여러 파티션들에 할당되어 데이터를 가져갈 수 있다. 그리고 1개의 파티션은 최대 1개의 컨슈머에만 할당이 가능하다.
* 따라서 컨슈머 그룹의 컨슈머 개수는 토픽의 파티션 개수보다 같거나 작아야 한다. 예를 들어 3개의 파티션을 갖는 토픽을 효과적으로 처리하려면 3개 이하로 이뤄진 컨슈머 그룹으로 운영해야 한다.
* 현재 운영하고 있는 토픽의 데이터가 어디에 적재되는지, 어떻게 처리되는지 파악하여 컨슈머 그룹을 최대한 나누는 것이 좋다.
  * 예를 들어 elastic search와 hadoop에 데이터를 동기로 적재하던 파이프라인을 카프카를 통해 각각의 저장소에 비동기로 적재하도록 변경할 수 있다. 이 때 컨슈머 그룹을 분리하지 않으면 한쪽 저장소에 장애가 발생했을 때 적재에 대해 지연이 발생할 수 있다.

#### 리밸런싱

* **컨슈머가 추가/제거되는 상황에 리밸런싱이 발생**한다.
* 컨슈머 그룹의 컨슈머 중 일부에 장애가 발생(ex. 연결 불가)하면 해당 컨슈머에 할당되어 있던 파티션은 장애가 발생하지 않은 컨슈머에 소유권이 넘어간다.
* 언제든지 발생할 수 있기 때문에 데이터 처리 중 발생한 리밸런싱에 대응하는 코드를 컨슈머 클라이언트에 작성해두어야 한다.
* 파티션의 소유권을 재할당할 때 다른 컨슈머들이 토픽의 데이터를 읽을 수 없으므로 리밸런싱은 자주 발생하면 안된다.
* 카프카 브로커 중 한 프로세스가 **그룹 코디네이터**가 되어 컨슈머 그룹의 컨슈머 추가/삭제를 감지하고 리밸런싱을 발동시킨다.

#### 커밋

* 컨슈머가 브로커로부터 어느 데이터까지 가져갔는지 **커밋**을 통해 기록한다.
* 카프카 브로커의 내부 토픽인 `__consumer_offsets` 에 특정 토픽의 파티션을 어떤 컨슈머 그룹이 어느 레코드까지 읽었는지에 대한 정보가 기록된다. 만약 이 토픽에 오프셋 커밋이 기록되지 않는다면 데이터를 중복해서 처리할 수 있기 때문에 **컨슈머 애플리케이션은 오프셋 커밋이 정상적으로 처리되었는지 검증해야 한다.**
* **비명시 오프셋 커밋**이란 일정 간격마다 오프셋을 커밋하도록 하는 것이다. 이 방식은 편리하지만 리밸런싱 또는 컨슈머 강제 종료 시&#x20;





### Kafka Client로 컨슈머 구현하기

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





