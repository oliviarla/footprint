# 함수형 엔드포인트

## 개념

* 들어오는 요청을 라우팅하여 처리하고 결과 값을 응답하는 모든 과정을 하나의 함수 체인에서 수행하도록 한다.

## HandlerFunction

* HandlerFunction은 요청 정보를 `ServerRequest` 형태로 입력받고 응답을 `Mono<ServerResponse>` 형태로 반환하는 handle 메서드를 가진다.
* Request를 처리하는 역할을 한다.
* ServerRequest
  * HTTP headers, method, URI, query parameters에 접근할 수 있는 메서드를 제공한다.
  * HTTP body 정보에 접근하기 위한 메서드를 제공한다.
* ServerResponse
  * HandlerFunction 또는 HandlerFilterFunction에서 반환되는 HTTP Response를 표현한다.
  * BodyBuilder, HeadersBuilder를 통해 HTTP Response body와 header 정보를 추가할 수 있다.

## RouterFunction

* 요청이 들어오면 HandlerFunction으로 라우팅해준다.
* 요청 처리를 위한 HandlerFunction을 반환한다.
* 다음은 각각의 request를 처리하기 위한 라우트를 추가하는 예제이다.

```java
@Configuration("bookRouterV1")
public class BookRouter {
    @Bean
    public RouterFunction<?> routeBookV1(BookHandler handler) {
        return route()
                .POST("/v1/books", handler::createBook)
                .PATCH("/v1/books/{book-id}", handler::updateBook)
                .GET("/v1/books", handler::getBooks)
                .GET("/v1/books/{book-id}", handler::getBook)
                .build();
    }
}
```

* 다음은 request를 처리하는 handler 클래스의 예제이다.
  * bodyToMono는 HTTP request body를 Mono로 변환해주는 메서드이다.

```java
@Component("bookHandlerV1")
public class BookHandler {
    private final BookMapper mapper;

    public BookHandler(BookMapper mapper) {
        this.mapper = mapper;
    }

    public Mono<ServerResponse> createBook(ServerRequest request) {
        return request.bodyToMono(BookDto.Post.class)
                .map(post -> mapper.bookPostToBook(post))
                .flatMap(book ->
                        ServerResponse
                                .created(URI.create("/v1/books/" + book.getBookId()))
                                .build());
    }
    // ...
}
```

## Validator

* request body의 유효성을 검증하기 위해 Spring의 Validator 인터페이스를 구현한 Custom Validator를 사용할 수 있다.
* 아래와 같이 Validator 인터페이스를 구현한 BookValidator를 **doOnNext 오퍼레이터 내부에서 사용**해 request body의 유효성을 검증할 수 있다.

```java
@Component("bookHandlerV2")
public class BookHandler {
    private final BookMapper mapper;
    private final BookValidator validator;

    public BookHandler(BookMapper mapper, BookValidator validator) {
        this.mapper = mapper;
        this.validator = validator;
    }

    public Mono<ServerResponse> createBook(ServerRequest request) {
        return request.bodyToMono(BookDto.Post.class)
                .doOnNext(post -> this.validate(post))
                .map(post -> mapper.bookPostToBook(post))
                .flatMap(book ->
                        ServerResponse
                                .created(URI.create("/v2/books/" + book.getBookId()))
                                .build());
    }
    // ...
}
```

* Custom Validator를 이용해 직접 유효성 검증 로직을 작성하는 방식은 비즈니스 코드와 유효성 검증을 위한 코드가 뒤섞여 있어 바람직한 방식이 아니다.
* 유효성 검증을 Spring에서 제공하는 표준 Bean Validation을 통해 수행하도록 하면 비즈니스 로직과 엮일 필요 없이 간단히 구현할 수 있다.
* Validator 구현체를 의존성 주입으로 받아 사용할 수 있다. 이 때 내부적으로 Hibernate Validator 구현체가 주입된다.
* Handler에서 사용하려면, 앞서 소개한 Custom Validator와 같이 doOnNext 오퍼레이터에서 validate 메서드를 호출하면 된다.
* 아래는 표준 Bean Validation을 사용할 수 있는 Custom Validator 클래스이다. Validator 인터페이스를 의존성 주입받는 것을 볼 수 있다. Validator 인터페이스는 Spring 또는 Javax 에서 제공하는 인터페이스를 사용해야 한다.

```java
@Component("bookValidatorV3")
public class BookValidator<T> {
    private final Validator validator;

    public BookValidator(@Qualifier("springValidator") Validator validator) {
        this.validator = validator;
    }

    public void validate(T body) {
        Errors errors =
                new BeanPropertyBindingResult(body, body.getClass().getName());

        this.validator.validate(body, errors);

        if (!errors.getAllErrors().isEmpty()) {
            onValidationErrors(errors);
        }
    }

    private void onValidationErrors(Errors errors) {
        log.error(errors.getAllErrors().toString());
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, errors.getAllErrors()
                .toString());
    }
}
```
