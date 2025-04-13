# 카프카 설치 및 실습

## 카프카 브로커 클러스터 구성

* docker compose를 사용하여 간단히 로컬에 카프카 브로커 클러스터를 구성할 수 있다.
*   Kafka 브로커 실행을 위해서는 여러 속성을 설정해주어야 한다.

    > 카프카에서 나타내는 속성 이름과 docker compose에서 나타내는 속성 이름이 상이하지만 여기서는 카프카에서 나타내는 속성 이름을 기준으로 설명한다.

    * `broker.id`: 실행하는 카프카 브로커의 번호를 직접 부여해주어야 한다. 브로커들을 구분하기 위해 부여하는 것이므로 다른 브로커와 겹치면 안된다.
    * `listeners`: 카프카 브로커의 통신을 위해 열어둘 인터페이스 IP, port, 프로토콜을 설정할 수 있다.
    * `advertised.listeners`: 카프카 클라이언트 또는 카프카 커맨드 라인 툴에서 접속 시 사용하는 IP, port 정보
    * `listener.security.protocol.map`: 보안 설정 시 사용할 프로토콜 매핑 지정
    * `num.network.threads`: 네트워크 처리를 위한 스레드 개수
    * `num.io.threads` : 카프카 브로커 내부에서 사용할 스레드 개수
    * `log.dirs` : 데이터를 저장할 디렉토리 위치
    * `num.partitions` : 토픽 당 파티션 개수를 지정할 수 있으며 파티션이 많아질수록 데이터 소비를 병렬로 많이 처리할 수 있게 된다.
    * `log.retention.ms` : 카프카 브로커가 저장한 파일이 삭제되기 까지 걸리는 시간 지정, -1로 설정하면 영원히 삭제되지 않는다.
    * `log.segment.bytes`: 카프카 브로커가 저장할 파일의 최대 크기
    * `log.retention.check.interval.ms`: 카프카 브로커가 저장한 파일을 삭제하기 위해 체크하는 간격(ms)

```yaml
networks:
  kafka_network:

volumes:
  Kafka00:
    driver: local
  Kafka01:
    driver: local
  Kafka02:
    driver: local

services:
  Kafka00Service:
    image: bitnami/kafka:3.5.1-debian-11-r44
    restart: unless-stopped
    container_name: Kafka00Container
    ports:
      - '10000:9094'
    environment:
      - KAFKA_CFG_BROKER_ID=0
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka00Service:9092,EXTERNAL://127.0.0.1:10000
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
    networks:
      - kafka_network
    volumes:
      - "Kafka00:/bitnami/kafka"
  Kafka01Service:
    image: bitnami/kafka:3.5.1-debian-11-r44
    restart: always
    container_name: Kafka01Container
    ports:
      - '10001:9094'
    environment:
      - KAFKA_CFG_BROKER_ID=1
      - KAFKA_CFG_NODE_ID=1
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka01Service:9092,EXTERNAL://127.0.0.1:10001
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
    networks:
      - kafka_network
    volumes:
      - "Kafka01:/bitnami/kafka"
  Kafka02Service:
    image: bitnami/kafka:3.5.1-debian-11-r44
    restart: always
    container_name: Kafka02Container
    ports:
      - '10002:9094'
    environment:
      - KAFKA_CFG_BROKER_ID=2
      - KAFKA_CFG_NODE_ID=2
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka02Service:9092,EXTERNAL://127.0.0.1:10002
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
    networks:
      - kafka_network
    volumes:
      - "Kafka02:/bitnami/kafka"

  KafkaWebUiService:
    image: provectuslabs/kafka-ui:latest
    restart: always
    container_name: KafkaWebUiContainer
    ports:
      - '8080:8080'
    environment:
      - KAFKA_CLUSTERS_0_NAME=Local-Kraft-Cluster
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=Kafka00Service:9092,Kafka01Service:9092,Kafka02Service:9092
      - DYNAMIC_CONFIG_ENABLED=true
      - KAFKA_CLUSTERS_0_AUDIT_TOPICAUDITENABLED=true
      - KAFKA_CLUSTERS_0_AUDIT_CONSOLEAUDITENABLED=true
      #- KAFKA_CLUSTERS_0_METRICS_PORT=9999
    depends_on:
      - Kafka00Service
      - Kafka01Service
      - Kafka02Service
    networks:
      - kafka_network
```

* 8080 포트에 브라우저로 접근하면 WebUi 서비스에 접근할 수 있다.

<figure><img src="../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 토픽 생성

* 도커 컨테이너에 exec으로 접근하여 `/opt/bitnami/kafka/bin` 디렉토리에서 아래와 같은 명령을 수행하면 토픽이 생성된다.

```
./kafka-topics.sh --create --bootstrap-server localhost:9092 --topic hello.kafka
```

* 파티션 개수와 복제 개수, 토픽 데이터 유지 기간 옵션을 지정하여 토픽을 생성할 수도 있다.
  * `partitions`: 최소 개수는 1개이며, 옵션을 사용하지 않으면 카프카 브로커 구동 시 설정했던 `num.partitions` 옵션대로 설정된다.
  * `replication-factor`: 토픽의 파티션을 복제할 개수를 지정하며 1으로 지정하면 복제를 하지 않는다는 의미이고 2로 지정하면 복제본 1개를 사용하겠다는 의미이다.
  * `config`: kafka-topics.sh 명령에 포함되지 않은 추가적인 설정을 할 수 있다. retention.ms의 경우 토픽의 데이터를 유지하는 기간을 의미한다.

```
./kafka-topics.sh --create --bootstrap-server localhost:9092 \
--partitions 3 --replication-factor 1 --config retention.ms=172800000 \
--topic hello.kafka
```

## 토픽 조회

* `--list` 인자를 주어 카프카 브로커 클러스터가 가지는 토픽들의 이름을 조회할 수 있다.

```
./kafka-topics.sh --bootstrap-server localhost:9092 --list
```

<figure><img src="../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 자세한 내용을 조회하려면 `--describe --topic <토픽이름>` 을 인자로 주면 된다.

```
./kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic hello.kafka
```

<figure><img src="../../.gitbook/assets/image (15) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 토픽 옵션 수정

* 파티션 개수 변경을 위해서는 아래 명령어를 수행하면 된다. describe를 통해 확인해보면 실제로 파티션 개수가 변경된 것을 확인할 수 있다.

```
./kafka-topics.sh --bootstrap-server localhost:9092 --topic hello.kafka --alter --partitions 4
```

<figure><img src="../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 토픽 삭제를 위한 리텐션 기간 변경을 위해서는 아래 명령어를 수행하면 된다.

```
./kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name hello.kafka --alter --add-config retention.ms=86400000
```

<figure><img src="../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 토픽에 데이터 넣기

* 토픽에 넣는 데이터는 **레코드**라고 부르며 key-value 형태로 이루어진다. 아래와 같이 키가 없는 레코드를 입력할 수도 있고 키와 값의 구분자를 입력해 key-value 형태의 레코드를 입력할 수도 있다.

```
$ ./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic hello.kafka
> hello
> kafka
```

```
$ ./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic hello.kafka \
  --property "parse.key=true" --property "key.separator=:"
> key1:no1
> key2:no2
```

* 이렇게 전달된 레코드는 토픽의 파티션에 저장된다.&#x20;
* 기본 파티셔너를 사용하는 경우, 프로듀서가 파티션으로 레코드를 전달할 때 key가 존재한다면 해시 값에 따라 적절한 파티션으로 이동하고, key가 존재하지 않는다면 라운드 로빈으로 파티션 중 하나에 할당된다. 이러한 파티션 할당 방식은 커스텀하게 파티셔너를 구현해 변경할 수 있다.

## 토픽의 데이터 받기

* 아래 명령을 사용해 컨슈머를 통해 토픽의 데이터를 읽어올 수 있다. `--from-beginning` 옵션을 통해 토픽에 들어온 레코드를 처음부터 조회할 수 있다.

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello.kafka \
--from-beginning
```

* `property` 옵션을 통해 키 출력, 키 구분자 여부도 설정할 수 있다.
* \--group 옵션을 통해 컨슈머 그룹을 설정할 수 있다. 컨슈머 그룹은 1개 이상의 컨슈머로 구성되며 조회한 토픽의 메시지에 대해 커밋을 한다. 커밋이란 컨슈머가 특정 레코드까지 처리를 완료했음을 알리기 위해 레코드의 오프셋 번호를 카프카 브로커에 저장하는 것이다.

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello.kafka \
--property print.key=true --property key.separator="-" --group hello-group --from-beginning
```

## 컨슈머 그룹 확인

* 아래 명령을 사용해 존재하는 컨슈머 그룹들을 확인할 수 있다.

```
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

* 아래 명령을 사용해 특정 컨슈머 그룹에 대한 정보를 확인할 수 있다.
  * `CURRENT-OFFSET`: 컨슈머 그룹이 받고 있는 토픽의 파티션에 존재하는 가장 최신 오프셋이 몇 번인지 나타낸다.
  * `LOG-END-OFFSET`: 컨슈머 그룹의 컨슈머가 어느 오프셋까지 커밋했는지 알 수 있다.
  * `LAG`: 컨슈머 그룹이 토픽의 파티션에 있는 데이터를 가져가는 데에 얼마나 지연이 발생했는지 확인하기 위한 지표로, 컨슈머 그룹이 커밋한 오프셋과 파티션의 최신 오프셋 간의 차이를 나타낸다.

```
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group hello-group --describe
```

<figure><img src="../../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 간편 테스트

* 카프카 클러스터 설치 후 잘 동작하는지 확인해보기 위한 메시지를 자동으로 produce하고 consume하기 위해 아래 명령을 사용할 수 있다.
* `--max-message` 옵션을 사용해 테스트할 메시지의 개수를 정할 수 있으며, 수행 시 가장 마지막줄에는 수행 결과의 통계값이 출력된다.

```
./kafka-verifiable-producer.sh --bootstrap-server localhost:9092 --max-message 10 --topic verify-test
```

* producer에서 발행한 메시지를 조회할 수 있다. 여기서는 consume한 메시지 개수 등의 정보가 출력된다.

```
./kafka-verifiable-consumer.sh --bootstrap-server localhost:9092 --topic verify-test \
--group-id test-group
```

## 토픽 데이터 제거

* delete.json 파일을 먼저 구성하고, 해당 json 파일을 기반으로 명령을 보내 토픽의 특정 데이터를 제거할 수 있다.

```json
{
    "partitions": [
        {
            "topic": "test,
            "partition": 0,
            "offset": 50
        }
    ],
    "version": 1
}
```

```
./kafka-delete-records.sh --bootstrap-server localhost:9092 --offset-json-file delete.json
```
