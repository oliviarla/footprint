---
description: 여러 명령어들을 원자성있게 실행하도록 보장하는 방법
---

# Transaction

## 개념

* 다른 DB의 트랜잭션 개념과 다르게 rollback을 지원하지 않는다. ([출처](https://redis.io/docs/interact/transactions/#what-about-rollbacks)) 응답 속도가 중요한 캐시 시스템이기 때문에 롤백을 지원하는 것은 적절하지 않다고 판단한 모양이다.
* 따라서 트랜잭션 중간에 에러가 난 명령어가 있다면 실패한 명령어에 대해서만 에러 메시지를 반환하고, 나머지 명령어들은 모두 수행된다.
* Redis에서 말하는 트랜잭션이란 여러 명령어들의 묶음을 실행하는 사이에 다른 명령어가 실행되지 않도록(원자성) 보장할 뿐이다.

## 트랜잭션 활용법

### MULTI

* Redis 트랜잭션은 `MULTI` 명령을 사용하여 시작한다. 이 명령은 항상 OK로 응답한다.
* 이후 사용자가 여러 명령을 추가하면, Redis는 이 명령어들을 대기열에 넣는다.
* `EXEC`을 호출되면 대기열에 존재하던 모든 명령이 실행된다.
* `DISCARD`를 호출하면 트랜잭션 대기열을 모두 제거하고 트랜잭션을 종료한다.

```
> MULTI
OK
> INCR foo
QUEUED
> INCR bar
QUEUED
> EXEC
1) (integer) 1
2) (integer) 1
```

### WATCH

* `WATCH` 명령은 `EXEC` 명령이 호출되기 전까지 특정 키의 변경 사항을 감지하기 위해 모니터링한다.
* 만약 `EXEC`이 호출되기 전 하나 이상의 감시된 키가 수정되면 전체 트랜잭션이 중단되고, EXEC는 Null 응답(아무응답 없음)을 반환하여 트랜잭션이 실패했음을 알린다.
*   이 명령어를 사용해 CAS(check-and-set) 연산처럼 수행할 수 있다.

    * 예를 들어 아래와 같이 WATCH 명령으로 mykey를 잡고 있는데 EXEC 이전에 mykey에 해당하는 value 값이 변할 경우 트랜잭션이 실패하게 된다.

    ```
    WATCH mykey
    val = GET mykey
    val = val + 1 // 값 변경
    MULTI
    SET mykey $val
    EXEC
    ```

## Spring Data Redis에서 트랜잭션 사용하기

* 트랜잭션을 지원하지만, 트랜잭션의 모든 명령들을 하나의 커넥션에서만 수행됨을 보장하지 않는다.
* 즉, 트랜잭션 내에 읽기 명령, 쓰기 명령을 구분하고 커넥션을 나누어 수행한다.
* 하나의 커넥션으로 여러 데이터를 조회하고 싶다면, SessionCallback을 사용하거나 pipelining 기능을 이용하면 된다.

```java
//execute a transaction
List<Object> txResults = redisTemplate.execute(new SessionCallback<List<Object>>() {
  public List<Object> execute(RedisOperations operations) throws DataAccessException {
    operations.multi();
    operations.opsForSet().add("key", "value1");

    // This will contain the results of all operations in the transaction
    return operations.exec();
  }
});
```
