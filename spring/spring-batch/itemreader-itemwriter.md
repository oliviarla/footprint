# ItemReader/ItemWriter

## ItemReader

* 데이터를 읽어오는 read 메서드를 가진 인터페이스이다.

### ItemStream 인터페이스

* 보통 ItemReader 구현체는 ItemStream 인터페이스도 함께 구현한다.
* **주기적으로 상태를 저장하고 오류가 발생하면 해당 상태에서 복원**하기 위한 마커 인터페이스 역할을 한다.
* 작업 도중 장애/실패가 발생하는 경우, ExecutionContext에 저장되어 있던 기존 위치를 읽어 해당 위치부터 다시 실행할 수 있도록 한다.

### 분할 처리

* 배치 데이터의 경우 실시간 처리가 어려운 대용량 데이터를 주로 다루는데, 이 데이터를 한꺼번에 메모리에 올릴 수 없다. 따라서 분할 처리가 필요하다.
* Spring Batch는 이를 지원하기 위해 CursorItemReader, PagingItemReader를 제공한다.
  * Cursor 방식은 Database와 커넥션을 맺은 후, Cursor를 한칸씩 옮기면서 지속적으로 데이터를 조회한다.
  * Paging 방식은 한번에 10개 (혹은 개발자가 지정한 PageSize)만큼 데이터를 조회한다.

### JdbcCursorItemReader

* 동작 방식
  * Database에서 제공하는 Cursor 기능을 사용하여 데이터를 하나씩 조회해오는 방식이다.
  * JdbcCursorItemReader는 JDBC ResultSet 기반으로 하나의 SQL을 실행한 뒤 커서를 이동하며 한 행씩 읽는다.
  * `read()` 메소드는 데이터를 하나씩 가져와 ItemWriter로 데이터를 전달하고, 다음 데이터를 가져온다.
* 연결 유지 특성
  * DB connection을 Step 시작부터 종료까지 유지한다. 즉, 하나의 connection으로 Batch가 끝날 때까지 사용된다.
  * 이를 통해 reader & processor & writer가 Chunk 단위로 수행되고 주기적으로 Commit 된다.
  * Chunk가 커밋되더라도 커서는 Step 단위로 리소스이므로 ResultSet이 계속 열려있는 상태가 된다.
  * 장시간 트랜잭션을 유지하는 경우, DB 서버 메모리를 많이 점유하고 락 경합이 발생하는 등 문제가 발생할 수 있다.
  * Database와 애플리케이션 간의 connection이 끊어지지 않도록 SocketTimeout을 충분히 큰 값으로 설정해야 한다.
* 성능
  * 재시작 시 가장 마지막에 읽던 row에 도달하기 위해 기존에 읽어낸 row들을 skip해야 한다. 따라서 Paging 방식에 비해 재시작 비용이 든다.
  * 기본적인 읽기 성능은 페이징 방식에 비해 뛰어나기 때문에 대량의 데이터가 아닌 멀티쓰레드 환경이 아닌 곳에서 사용하기 적합하다고 한다.

### JdbcPagingItemReader

* 동작 방식
  * 페이지 크기(pageSize) 단위로 LIMIT / OFFSET 또는 키 기반 조건의 SQL문을 여러 번 실행하여 데이터를 조회한다.
  * pageSize를 명시하면 Spring Batch 내에서 `offset`과 `limit`을 자동으로 생성해 쿼리한다.
  * 페이지 단위로 트랜잭션 경계가 자연스럽게 분리된다.
  * 한 페이지를 읽을때마다 connection을 맺고 끊기 때문에, 아무리 많은 데이터라도 타임아웃과 부하 없이 수행될 수 있다.
  * 커밋 후 다음 페이지를 새 트랜잭션으로 조회하므로 장시간 트랜잭션 부담이 적다.
  * Hibernate, JPA 등 영속성 컨텍스트가 필요한 Reader 사용시 fetchSize와 ChunkSize는 같은 값을 유지해야한다.
  * 각 페이지마다 새로운 쿼리를 실행하므로 정렬된 데이터에서 페이징으로 조회해야 한다. 안그러면 데이터를 중복 조회할 수 있다.
* 성능
  * 재시작 시에 가장 마지막에 조회한 키 또는 페이지 정보를 읽어 해당 부분부터 조회한다. 따라서 오버헤드가 없다.
  * 운영 안정성을 위해서는 커서 방식보다 이 방식이 더 적절하다.

## ItemWriter

* 청크 단위로 축적된 데이터를 한 번에 쓴다. 이는 어플리케이션과 데이터베이스 간에 데이터를 주고 받는 회수를 최소화하여 성능을 높이기 위함이다.
