# MirrorMaker

## 개념

* 서로 다른 두 카프카 클러스터 간에 토픽을 복제하는 애플리케이션이다.
* 모든 토픽 데이터를 그대로 복제하는 것은 클러스터 간의 파티셔닝 전략과 개수가 다를 수도 있는 등의 문제로 인해 직접 프로듀서와 컨슈머를 사용하여 복제를 구현하기는 어렵다.
* 미러메이커2는 토픽의 데이터, 설정 등을 동기화하며 커넥터로 사용 가능하다.
* 또한 단방향, 양방향 복제 기능, ACL 복제, 새 토픽 자동 감지 기능 등을 제공한다.
* 복제된 토픽에는 클러스터 이름이 접두사로 붙게 된다. 예를들어 A 클러스터의 test 토픽이 B 클러스터로 복제된다면 A.test 토픽이 생성될 것이다.

## 사용 방법

* connect-mirror-maker.properties 파일을 통해 카프카 클러스터, 토픽, 복제 시 사용할 내부 토픽을 설정해야 한다.
* 아래는 클러스터 A의 test 토픽을 클러스터 B로 복제하기 위한 설정 파일이다.

```properties
cluster = A, B

A.bootstrap.servers = a-kafka:9092
B.bootstrap.servers = b-kafka:9092

A->B.enabled = true
A->B.topics = test

B->A.enabled = false
B->A.topics = .*

# 신규 생성된 토픽의 복제 개수를 지정한다.
replication.factor = 1

# 토픽 복제에 필요한 데이터를 저장하는 내부 토픽의 복제 개수를 지정한다.
checkpoints.topic.replication.factor = 1
heartbeats.topic.replication.factor = 1
offset-syncs.topic.replication.factor = 1
offset.storage.replication.factor = 1
status.storage.replication.factor = 1
config.storage.replication.factor = 1
```

* 아래 명령을 통해 미러메이커2를 실행할 수 있다.

```
bin/connect-mirror-maker.sh config/connect-mirror-maker.properties
```
