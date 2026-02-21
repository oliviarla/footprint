# 스트리밍 데이터 처리

## 스트리밍 처리 방법

* Server-Sent Events를 이용해 데이터를 스트리밍할 수 있다.
* HTTP get 요청을 처리하는 메서드가 Flux 타입을 반환하도록 하고, Content Type을 `text/event-stream` 으로 지정해 라우팅하면 된다.
* 다음은 DB로부터 모든 Book 테이블의 레코드를 읽어와 2초마다 emit하도록 하고, 이를 스트리밍하여 받을 수 있도록 API를 제공하는 예제이다.

```java
public Flux<Book> streamingBooks() {
    return template
            .select(Book.class)
            .all()
            .delayElements(Duration.ofSeconds(2L));
}
```

```java
@Bean
public RouterFunction<?> routeStreamingBook(BookService bookService,
                                            BookMapper mapper) {
    return route(RequestPredicates.GET("/v11/streaming-books"),
            request -> ServerResponse
                    .ok()
                    .contentType(MediaType.TEXT_EVENT_STREAM)
                    .body(bookService
                            .streamingBooks()
                            .map(book -> mapper.bookToResponse(book)),
			                    BookDto.Response.class));
}
```

* 스트리밍 받는 WebClient의 경우 bodyToFlux()를 이용해 Flux 형태로 응답을 받아 처리할 수 있다.

```java
WebClient webClient = WebClient.create("<http://localhost:8080>");
Flux<BookDto.Response> response =
        webClient
                .get()
                .uri("/v11/streaming-books")
                .retrieve()
                .bodyToFlux(BookDto.Response.class);

response.subscribe(book -> {
            log.info("bookId: {}", book.getBookId());
            log.info("description: {}", book.getDescription());
        },
        error -> log.error("# error happened: ", error));
```
