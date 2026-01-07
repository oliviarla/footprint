# 트랜잭션

## 트랜잭션 활성화 조건

* Replica Set 또는 Sharded Cluster 환경에서만 가능
  * Standalone인 경우&#x20;
* WiredTiger 스토리지 엔진에 한해 동작

## ReadConcern / WrtieConcern

* 어떤&#x20;

## Snapshot Isolation

### 개념

* 트랜잭션 **시작 시점의 데이터 스냅샷**을 기준으로 작업을 수행하는 격리 수준
* 트랜잭션 내의 모든 읽기는 동일한 시점의 데이터를 기준으로 수행된다. 이로 인해 `REPEATABLE READ` 격리 수준이 제공된다.
* 여러 트랜잭션이 동시에 특정 데이터에 접근하여 수정하는 경우, 쓰기 충돌이 발생하여 트랜잭션이 실패할 수 있다.

### 장점



### 단점

* Write Skew

## 롤백



## 분산 트랜잭션

* WiredTiger&#x20;
* Sharded Cluster 환경의 경우 트랜잭션 사용 시 2PC를 사용한다. 즉 Coordinator가 각 샤드에 prepare 요청을 보내고 모두 가능하다는 응답을 받은 경우에만 트랜잭션을 커밋한다. 만약 하나의 노드라도 실패 응답을 보내거나 응답이 없다면 롤백한다.







