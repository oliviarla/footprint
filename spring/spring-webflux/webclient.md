# WebClient

## 개념 및 예시

* Spring 5 부터 지원하는 Non-Blocking HTTP Request를 위한 클래스이다.
* 내부적으로 HTTP 클라이언트 라이브러리(기본적으로 Reactor Netty 사용)에 위임하여 처리한다.
* 아래는 HTTP patch 요청을 보내는 예시이다. 메서드 체이닝을 통해 간결하게 HTTP 요청을 보내고 Mono 타입으로 응답을 받도록 한다.

```java
WebClient webClient = WebClient.create("<http://localhost:8080>");
Mono<BookDto.Response> response =
        webClient
                .patch()
                .uri("/v10/books/{book-id}", 20)
                .bodyValue(requestBody)
                .retrieve()
                .bodyToMono(BookDto.Response.class);

response.subscribe(book -> {
    log.info("bookId: {}", book.getBookId());
    log.info("description: {}", book.getDescription());
});
```

* 아래는 HTTP get 요청을 보내는 예시이며, Java Collection 타입의 response body를 Flux 타입으로 반환받는다.

```java
Flux<BookDto.Response> response =
        WebClient
                .create("<http://localhost:8080>")
                .get()
                .uri(uriBuilder -> uriBuilder
                        .path("/v10/books")
                        .queryParam("page", "1")
                        .queryParam("size", "10")
                        .build())
                .retrieve()
                .bodyToFlux(BookDto.Response.class);

response
        .map(book -> book.getTitleKorean())
        .subscribe(bookName -> log.info("book name: {}", bookName));
```

## Timeout 처리

* HttpClient 객체를 생성하여 WebClient에 HttpClientConnector를 설정할 수 있다.
* HttpClient에서는 아래와 같은 메서드들을 이용해 Timeout 설정을 할 수 있다.
  * option() 메서드로 Connection Timeout을 설정할 수 있다.
  * responseTimeout() 메서드로 응답을 수신하기까지의 Timeout을 설정할 수 있다.
  * doOnConnected() 메서드를 통해 특정 시간동안 읽을 데이터가 없거나 쓰기 작업이 오래걸리는 경우에 Timeout을 설정할 수 있다.
* WebClient의 빌더 패턴에서 clientConnector 메서드를 이용해 HttpClient를 주입할 수 있다. ReactorClientHttpConnector는 Reactor Netty에서 제공하는 클래스이다.

```java
HttpClient httpClient =
        HttpClient
                .create()
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 500)
                .responseTimeout(Duration.ofMillis(500))
                .doOnConnected(connection ->
                    connection
                            .addHandlerLast(
                                    new ReadTimeoutHandler(500,
                                                        TimeUnit.MILLISECONDS))
                            .addHandlerLast(
                                    new WriteTimeoutHandler(500,
                                                        TimeUnit.MILLISECONDS)));

Flux<BookDto.Response> response =
        WebClient
                .builder()
                .baseUrl("<http://localhost:8080>")
                .clientConnector(new ReactorClientHttpConnector(httpClient))
                .build()
                .get()
                .uri(uriBuilder -> uriBuilder
                        .path("/v10/books")
                        .queryParam("page", "1")
                        .queryParam("size", "10")
                        .build())
                .retrieve()
                .bodyToFlux(BookDto.Response.class);

response
        .map(book -> book.getTitleKorean())
        .subscribe(bookName -> log.info("book name2: {}", bookName));
```

## exchangeToMono / exchangeToFlux

* WebClient를 통해 얻은 response를 조작하기 위해 사용되는 메서드이다.
* 아래는 response가 CREATED인 경우에만 ResponseEntity를 반환하고, 나머지 경우 예외를 반환하도록 하는 예제이다.

```java
WebClient webClient = WebClient.create();
webClient
        .post()
        .uri("<http://localhost:8080/v10/books>")
        .bodyValue(post)
        .exchangeToMono(response -> {
            if(response.statusCode().equals(HttpStatus.CREATED))
                return response.toEntity(Void.class);
            else
                return response
                        .createException()
                        .flatMap(throwable -> Mono.error(throwable));
        })
        .subscribe(res -> {
            log.info("response status2: {}", res.getStatusCode());
            },
            error -> log.error("Error happened: ", error));
```
