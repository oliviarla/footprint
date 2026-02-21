# 예외 처리

## 예외 처리

* Spring MVC 기반에서 사용하는 @ExceptionHandler, @ControllerAdvice를 Spring WebFlux에서도 사용할 수 있다.
* 이 방식 외에도 Spring WebFlux 전용 예외 처리 기법이 존재한다.

#### onErrorResume

* 에러 이벤트가 발생했을 때 downstream으로 전파하지 않고 1. 대체 Publisher를 통해 에러 이벤트에 대한 대체 값을 emit하거나 2. 에러를 래핑한 또 다른 에러 이벤트를 발생시킨다.
* 처리할 Exception 클래스 타입과 대체를 위한 publisher의 sequence를 입력한다.

```java
return bookService.findBook(bookId)
                .flatMap(book -> ServerResponse
                        .ok()
                        .bodyValue(mapper.bookToResponse(book)))
                .onErrorResume(Exception.class, error -> ServerResponse
                        .badRequest()
                        .bodyValue(new ErrorResponse(HttpStatus.BAD_REQUEST,
                                error.getMessage())));
```

#### ErrorWebExceptionHandler

* GlobalExceptionHandler와 유사하지만 Mono 형태의 오퍼레이터 체인으로 예외를 처리할 수 있다.
* DefaultErrorWebExceptionHandler 빈이 기본적으로 등록되어 있으므로 커스텀하게 ErrorWebExceptionHandler 구현체를 빈으로 등록 시에는 `@Order(-2)` 로 우선순위를 높여주어야 한다.
* DataBufferFactory는 response body를 write하기 위해 필요한 객체이다.

```java
@Order(-2)
@Configuration
public class GlobalWebExceptionHandler implements ErrorWebExceptionHandler {
    private final ObjectMapper objectMapper;

    public GlobalWebExceptionHandler(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public Mono<Void> handle(ServerWebExchange serverWebExchange,
                             Throwable throwable) {
        return handleException(serverWebExchange, throwable);
    }

    private Mono<Void> handleException(ServerWebExchange serverWebExchange,
                                       Throwable throwable) {
        ErrorResponse errorResponse = null;
        DataBuffer dataBuffer = null;

        DataBufferFactory bufferFactory =
                                serverWebExchange.getResponse().bufferFactory();
        serverWebExchange.getResponse().getHeaders()
                                        .setContentType(MediaType.APPLICATION_JSON);

        if (throwable instanceof BusinessLogicException) {
            BusinessLogicException ex = (BusinessLogicException) throwable;
            ExceptionCode exceptionCode = ex.getExceptionCode();
            errorResponse = ErrorResponse.of(exceptionCode.getStatus(),
                                                exceptionCode.getMessage());
            serverWebExchange.getResponse()
                        .setStatusCode(HttpStatus.valueOf(exceptionCode.getStatus()));
        } else if (throwable instanceof ResponseStatusException) {
            ResponseStatusException ex = (ResponseStatusException) throwable;
            errorResponse = ErrorResponse.of(ex.getStatus().value(), ex.getMessage());
            serverWebExchange.getResponse().setStatusCode(ex.getStatus());
        } else {
            errorResponse = ErrorResponse.of(HttpStatus.INTERNAL_SERVER_ERROR.value(),
                                                            throwable.getMessage());
            serverWebExchange.getResponse()
                                    .setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR);
        }

        try {
            dataBuffer =
                    bufferFactory.wrap(objectMapper.writeValueAsBytes(errorResponse));
        } catch (JsonProcessingException e) {
            bufferFactory.wrap("".getBytes());
        }

        return serverWebExchange.getResponse().writeWith(Mono.just(dataBuffer));
    }
}
```
