# Spring Data R2DBC

## 개념 및 사용 방법

* RDB에 리액티브 프로그래밍 API를 제공하기 위한 specification이며, 클라이언트가 사용하기 위한 Service Provider Interface이다.
* R2DBC 라이브러리를 사용하면 RDB를 사용하더라도 완전한 Non-Blocking 애플리케이션을 구현할 수 있다.
* 엔티티 클래스에 정의된 매핑 정보로 테이블을 자동생성해주는 기능이 없기 때문에 직접 테이블 스크립트를 작성해주어야 한다.
* Auditing, Repository 기능을 활성화하기 위해 어노테이션을 추가해주어야 한다.

```java
@EnableR2dbcRepositories
@EnableR2dbcAuditing
```

* Repository를 생성하고 새로운 메서드를 정의하는 것은 Spring Data JPA와 유사하다.

```java
@Repository("bookRepositoryV5")
public interface BookRepository extends ReactiveCrudRepository<Book, Long> {
    Mono<Book> findByIsbn(String isbn);
}
```

## 엔티티 클래스 매핑

* @Id
  * 테이블의 기본 키에 해당하는 필드에 추가
* @Table
  * 클래스 레벨에 이름을 붙여 어떤 이름을 가진 테이블과 매핑할지 결정
* @CreatedDate, @LastModifiedDate
  * 레코드 생성/수정 날짜를 자동으로 테이블에 반영하기 위함
  * Auditing 기능을 활성화한 상태여야 한다.

## R2dbcEntityTemplate

* 기존 Spring Data JPA에서 제공하는 Repository를 사용하는 방식은 개발자들에게 널리 알려진 방법이다.
* Spring Data R2DBC에서는 가독성 좋은 SQL 쿼리문을 사용하는 것과 비슷하게, 여러 메서드를 체이닝해 원하는 데이터를 가져올 수 있도록 하는 R2dbcEntityTemplate를 제공한다.
* QueryDSL과 유사한 방식의 Query 생성 메서드 조합과 Entity 객체를 template에 전달하여 데이터베이스와 상호작용하게 된다.
* 메서드의 종류는 아래와 같이 나뉜다.
  * `Entrypoint` 메서드: 쿼리 시작 구문에 해당하는 select(), insert(), update(), delete() 등이 해당된다.
  * `Terminating` 메서드: select() 메서드와 함께 사용할 수 있으며 SQL문을 최종적으로 실행한다.
  * `Criteria` 메서드: SQL 연산자에 해당하는 and(String column), isNull(), not(Object o) 등이 해당된다.
* 아래는 Service 클래스에서 template을 이용해 데이터베이스 요청을 보내고 결과를 받는 예제이다.

```java
@Service("bookServiceV6")
@RequiredArgsConstructor
public class BookService {
    private final @NonNull R2dbcEntityTemplate template;

    public Mono<List<Book>> findBooks() {
        return template.select(Book.class).all().collectList();
    }
    
    public Mono<Book> findBook(long bookId) {
        return template.selectOne(query(where("BOOK_ID").is(bookId)), Book.class)
                .switchIfEmpty(Mono.error(new BusinessLogicException(
                                                ExceptionCode.BOOK_NOT_FOUND)));
    }
    
    public Mono<Book> saveBook(Book book) {
        return verifyExistIsbn(book.getIsbn())
                .then(template.insert(book));
    }
    
    private Mono<Void> verifyExistIsbn(String isbn) {
        return template.selectOne(query(where("ISBN").is(isbn)), Book.class)
                .flatMap(findBook -> {
                    if (findBook != null) {
                        return Mono.error(new BusinessLogicException(
                                ExceptionCode.BOOK_EXISTS));
                    }
                    return Mono.empty();
                });
    }
}
```

## Pagination

*   Repository 클래스에 메서드를 정의하는 방식

    * Pageable 클래스를 인자로 받아 특정 부분의 데이터만 Flux\<T> 형태로 얻을 수 있다.

    ```java
    @Repository("bookRepositoryV7")
    public interface BookRepository extends ReactiveCrudRepository<Book, Long> {
        Flux<Book> findAllBy(Pageable pageable);
    }
    ```

    * Flux 형태로 받은 데이터들을 Mono\<List\<T>> 로 변형하려면 collectList() 오퍼레이터를 사용하면 된다.

    ```java
    public Mono<List<Book>> findBooks(@Positive int page,
                                      @Positive int size) {
        return bookRepository
                .findAllBy(PageRequest.of(page - 1, size, Sort.by("memberId").descending()))
                .collectList();
    }
    ```
*   R2dbcEntityTemplate으로 메서드 체이닝하는 방식

    * limit(), offset(), sort() 등의 쿼리 빌드 메서드를 조합할 수 있다.
    * 다음 예제는 skip(), take() 오퍼레이터를 이용해 특정 수만큼 emit된 데이터를 건너뛰고, 그 뒤로 emit되는 데이터부터 특정 수만큼만 조회하도록 한다.

    ```java
    public Mono<List<Book>> findBooks(@Positive long page, @Positive long size) {
        return template
                .select(Book.class)
                .count()
                .flatMap(total -> {
                    Tuple2<Long, Long> skipAndTake = getSkipAndTake(total, page, size);
                    return template
                            .select(Book.class)
                            .all()
                            .skip(skipAndTake.getT1())
                            .take(skipAndTake.getT2())
                            .collectSortedList((Book b1, Book b2) ->
                                    (int) (b2.getBookId() - b1.getBookId()));
                });
    }

    private Tuple2<Long, Long> getSkipAndTake(long total, long movePage, long size) {
        long totalPages = (long) Math.ceil((double) total / size);
        long page = movePage > totalPages ? totalPages : movePage;
        long skip = total - (page * size) < 0 ? 0 : total - (page * size);
        long take = total - (page * size) < 0 ? total - ((page - 1) * size) : size;

        return Tuples.of(skip, take);
    }
    ```

    * 하지만 이 방식은 모든 데이터를 메모리에 가져온 후에 골라내는 것이므로, limit(), offset() 방식을 사용하는 것이 성능적으로 유리하다.

    ```java
    public Mono<List<Book>> findBooks(@Positive long page, @Positive long size) {
        return template
                .select(Book.class)
                .count()
                .flatMap(total -> {
                    long totalPages = (long) Math.ceil((double) total / size);
                    long validPage = Math.min(page, totalPages);
                    long offset = (validPage - 1) * size;

                    return template
                            .select(Book.class)
                            .matching(Query.query(Criteria.empty())
                                    .sort(Sort.by(Sort.Order.desc("bookId")))
                                    .limit((int) size)
                                    .offset((int) offset))
                            .all()
                            .collectList();
                });
    }
    ```
