# 내부 동작 방식

## 동작 흐름

> 클라이언트 요청부터 응답을 반환하기 까지 과정이 Mono/Flux의 Operator chain으로 구성된 하나의 Reactor Sequence로 구성되어 있다고 생각하면 된다.

* 클라이언트 요청이 들어오면 서버 엔진을 거쳐 HttpHandler로 전달된다.
* ServerWebExchange(ServerHttpRequest, ServerHttpResponse를 포함)를 생성하여 WebFilter 체인으로 전달한다.
* ServerWebExchange 객체는 WebFilter에서 전처리 과정을 거친다.
* 이후 DispatcherHandler(WebHandler 인터페이스의 구현체이며 DispatcherServlet과 유사함)에 전달되어 HandlerMapping List를 원본 Flux의 source로 전달받는다.
* ServerWebExchange 처리를 위한 Handler를 조회한 후 HandlerAdapter에 핸들러 호출을 위임한다.
* Controller 혹은 HandlerFunction 형태의 핸들러에서 요청을 처리한 후 응답 데이터를 포함한 Mono/Flux를 반환한다.
* 응답 데이터를 처리할 HandlerResultHandler에게 넘긴 후 최종 응답이 반환되도록 한다.

> 메서드가 호출을 통해 반환된 Reactor Sequence는 즉시 작업을 처리하지 않는다.

## 컴포넌트

* HttpHandler
  * http request, response를 처리하기 위한 handle 메서드 하나만을 가진다.
  * HttpWebHandlerAdapter 클래스는 이 인터페이스의 구현체로, ServerWebExchange 객체를 생성하여 WebHandler를 호출한다.
* WebFilter
  * 서블릿 필터와 유사하게 핸들러가 직접 요청을 처리하기 전, 보안, 세션 타임아웃 등의 전처리 과정을 여러 필터들을 거치며 수행되도록 한다.
  * 어노테이션 기반 요청 핸들러와 함수형 기반 요청 핸들러 모두에 적용된다.
* HandlerFilterFunction
  * 함수형 기반 요청 핸들러에만 적용할 수 있는 필터이다.
  * WebFilter 구현체는 스프링 빈으로 등록해야 하는 반면, HandlerFilterFunction 구현체는 함수형 기반 요청 핸들러에 함수 파라미터 형태로 전달되므로 스프링 빈에 등록하지 않아도 된다.
* DispatcherHandler
  * 객체 초기화 과정에서 ApplicationContext를 통해 스프링 빈으로 등록된 HandlerMapping, HandlerAdapter, HandlerResultHandler 컴포넌트를 찾아두고, 요청이 들어오면 알맞은 컴포넌트에 위임한다.
* HandlerMapping
  * getHandler 메서드를 제공하여 ServerWebExchange를 처리할 핸들러 객체를 얻을 수 있도록 한다.
* HandlerAdapter
  * HandlerMapping으로부터 얻은 핸들러를 직접 호출하여 얻은 결과인 `Mono<HandlerResult>` 를 반환한다.
