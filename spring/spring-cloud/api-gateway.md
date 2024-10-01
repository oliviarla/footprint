# API Gateway

* 클라이언트에서 직접 마이크로서비스의 API를 호출하는 대신 API Gateway와만 통신할 수 있도록 하여 마이크로서비스의 주소가 변경되는 등의 상황에 유연하게 대처할 수 있도록 한다.
* API Gateway를 사용함으로써 얻게되는 기능은 다음과 같다.
  * 인증 및 권한 부여
  * 마이크로서비스 검색 통합
  * 응답 캐싱
  * 일괄적인 정책 적용, 회로 차단기, QoS 다시 시도 가능
  * 속도 제한
  * 로드밸런싱
  * 로깅, 마이크로서비스 호출 경로 추적, 상관 관계
  * 헤더, 쿼리 문자열 등 요청 데이터 변환
  * IP 허용 목록을 기반으로 요청 처리

## Netflix Ribbon

* 클라이언트 사이드 로드밸런서로, 클라이언트 프로그램 안에서 이동하고자 하는 서비스의 주소값을 관리하는 프로그램이다.
* 비동기 처리가 잘 지원되지 않는다.
* 직접 주소를 사용해 API를 호출하는 대신 마이크로서비스의 이름을 통해 API를 호출할 수 있다.
* health check가 가능하다.
* Spring Boot 2.4부터 더이상 사용할 수 없는 maintainance 상태이다. 대신 Spring Cloud Loadbalancer를 사용하면 된다.

## Netflix Zuul

* Routing, API gateway 역할을 한다.
* Spring Boot 2.4부터 더이상 사용할 수 없는 maintainance 상태이다. 대신 Spring Cloud Gateway를 사용하면 된다.

### 예제

* 아래와 같이 아주 간단한 컨트롤러를 가진 WAS를 두 대 구동하여 eureka에 등록한다. First Service, Second Service를 각각 8081, 8082 포트에 구동하면 된다.
* 이 때 eureka client 의존성을 추가해주어야 한다.

```java
@RestController
@RequestMapping("/")
public class BasicController {
    @GetMapping("/welcome")
    public String welcome() {
        return "This is First Service.";
    }
}
```

```yaml
server:
    port: 8081

spring:
    application:
        name: my-first-service
eureka:
    client:
        register-with-eureka: false
        fetchRegistry: false
```

* 그리고 Zuul Service를 위한 어플리케이션을 구현한다. 이를 통해 `localhost:8000/first-service/welcome`으로 접속하면 `localhost:8081/welcome`과 동일한 결과가 반환된다.

```java
@SpringBootApplication
@EnableZuulProxy
public class ZuulServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulServiceApplication.class, args);
    }
}
```

```yaml
server:
    port: 8000

zuul:
    routes:
        first-service:
            path: /first-service/**
            url: http://localhost:8081
        second-service:
            path: /second-service/**
            url: http://localhost:8082
```

* 인증 서비스, 로깅 등의 비즈니스 로직을 수행하기 위해 필터를 등록할 수 있다. 아래는 로깅을 하기 위한 Zuul 필터의 예제이다.

```java
@Component
public class ZuulLoggingFilter extends ZuulFilter {
    Logger logger = LoggerFactory.getLogger(ZuulLoggingFilter.class);
    
    @Override
    public Object run() throws ZuulException {
        logger.info("************ printing logs: ");
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        logger.info("************ " + request.getRequestURI());
        
        return null;
    }
    
    @Override
    public String filterType() {
        return "pre";
    }
    
    @Override
    public int filterOrder() {
        return 1;
    }
    
    @Override
    public boolean shouldFilter() {
        return true;
    }
    
}
```

## Spring Cloud Gateway

* 네티 기반의 웹서버로 구현되어 있어 비동기 처리가 가능하다.

<figure><img src="../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

### 예제

* 앞서 Zuul 예제와 동일하게 두 대의 WAS를 띄워둔다.
* 그리고 api gateway를 위한 새로운 애플리케이션을 구현한다.
* 아래와 같이 yaml 파일을 작성하거나 RouteLocator 빈을 등록하여 라우트 정보를 설정할 수 있다.
  * Zuul과 달리 `localhost:8000/first-service/welcome`으로 접속하면 `localhost:8081/first-service/welcome` URI가 호출된다. 이를 변경하려면 RewritePath 필터를 추가하면 된다.
  * 필터를 등록하여 헤더를 추가하거나 원하는 로직을 작성할 수 있다.

```yaml
server:
    port: 8000
eureka:
    client:
        register-with-eureka: false
        fetch-registry: false
        service-url:
            defaultZone: http://localhost:8761/eureka

spring:
    application:
        name: apigateway-service
    # Spring Cloud Gateway 설정
    cloud:
        gateway:
            routes:
                - id: first-service
                  uri: http://localhost:8081/
                  predicates:
                    - Path=/first-service/**
                  filters:
                    - AddRequestHeader=first-request, header1
                    - AddResponseHeader=first-response, header2
                    # 
                    - RewritePath=/first-service/(?<segment>.*), /$\{segment}
                - id: second-service
                  uri: http://localhost:8082/
                  predicates:
                    - Path=/second-service/**
                  filters:
                    - AddRequestHeader=second-request, header1
                    - AddResponseHeader=second-response, header2
```

```java
@Configuration
public class FilterConfig {
    @Bean
    public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
            .route(r -> r.path("/first-service/**")
                .filters(f -> f.addRequestHeader("first-request", "header1")
                    .addResponseHeader("first-response", "header2"))
                .uri("http://localhost:8081/"))
            .route(r -> r.path("/second-service/**")
                .filters(f -> f.addRequestHeader("second-request", "header1")
                    .addResponseHeader("second-response", "header2"))
                .uri("http://localhost:8082/"))
            .build();
    }
}
```

### 커스텀 필터

* 커스텀 필터를 아래와 같이 구현할 수 있다. 이후, 커스텀 필터를 Route에 등록해주어야 한다.

```java
@Component
@Slf4j
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {

    public CustomFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest req = exchange.getRequest();
            ServerHttpResponse res = exchange.getResponse();
            
            log.info("Custom PRE filter: request uri -> {}", request.getId());
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                log.info("Custom POST filter: response code -> {}", response.getStatusCode());
            }));
        };
    }
    
    public static class Config {
        // put the configuration properties
    }
}


```

```
spring:
    cloud:
        gateway:
            routes:
                - id: first-service
                  uri: http://localhost:8081/
                  predicates:
                    - Path=/first-service/**
                  filters:
                    - CustomFilter
```

### 글로벌 필터

* 각 라우터 정보에 필터를 매번 등록하는 대신 전역적으로 적용되는 필터를 구현해 사용할 수 있다.
* 모든 필터가 수행되기 전에 먼저 수행되고, 모든 필터가 수행된 후에 글로벌 필터도 종료된다.

```java
@Component
@Slf4j
public class GlobalFilter extends AbstractGatewayFilterFactory<GlobalFilter.Config> {
    public GlobalFilter() {
        super(Config.class);
    }
    
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest req = exchange.getRequest();
            ServerHttpResponse res = exchange.getResponse();
            
            log.info("Global Filter base message: {}", config.getBaseMessage());
            
            if (config.isPreLogger()) {
                log.info("Global Filter Start: request id -> {}", request.getId());
            }
            
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isPreLogger()) {
                    log.info("Global Filter End: response code -> {}", response.getStatusCode());
                }
            }));
        };
    }
    
    @Data
    public static class Config {
        private String baseMessage;
        private boolean preLogger;
        private boolean postLogger;
    }
}
```

* 글로벌 필터를 아래와 같이  등록할 수 있다. 매개변수를 설정하여 필터 내부의 Config 객체에 이 매개변수 값을 매핑하기 때문에 필터에서 값을 사용할 수 있다.

```yaml
default-filters:
    - name: GlobalFilter
      args:
        baseMessage: Spring Cloud Gateway Global Filter
        preLogger: true
        postLogger: true
```

### 필터 순서 제어

* 필터의 적용 순서를 제어하기 위해 아래와 같이 OrderedGatewayFilter 객체를 생성해 반환할 수 있다.
* HIGHEST\_PRECEDENCE일 경우 필터가 가장 바깥쪽에 위치하게 되며, LOWEST\_PRECEDENCE일 경우 필터가 가장 안쪽에 위치하게 된다.

```java
@Component
@Slf4j
public class LoggingFilter extends AbstractGatewayFilterFactory<LoggingFilter.Config> {
    public LoggingFilter() {
        super(Config.class);
    }
    
    @Override
    public GatewayFilter apply(Config config) {
        return new OrderedGatewayFilter((exchange, chain) -> {
            ServerHttpRequest req = exchange.getRequest();
            ServerHttpResponse res = exchange.getResponse();
            
            log.info("Global Filter base message: {}", config.getBaseMessage());
            
            if (config.isPreLogger()) {
                log.info("Global Filter Start: request id -> {}", request.getId());
            }
            
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isPreLogger()) {
                    log.info("Global Filter End: response code -> {}", response.getStatusCode());
                }
            }));
        }, Ordered.LOWEST_PRECEDENCE);
    }
```

### 로드밸런서

* API gateway는 eureka 서버에 어떤 마이크로 서비스를 호출해야 할 지 확인한 후 해당 마이크로 서비스로 요청을 보내게 된다.
* 이 때 로드밸런싱을 하기 위해서는 라우트 정보에 직접 호스트와 포트 정보를 지정하는 대신, 서비스 이름을 지정하여 해당 서비스 이름을 가진 여러 WAS에 요청을 분산해 보낼 수 있도록 한다.

```yaml
server:
    port: 8000
eureka:
    client:
        register-with-eureka: true
        fetch-registry: true
        service-url:
            defaultZone: http://localhost:8761/eureka

spring:
    application:
        name: apigateway-service
    # Spring Cloud Gateway 설정
    cloud:
        gateway:
            routes:
                - id: first-service
                  uri: lb://MY-FIRST-SERVICE
                  predicates:
                    - Path=/first-service/**
                - id: second-service
                  uri: lb://MY-SECOND-SERVICE
                  predicates:
                    - Path=/second-service/**
```
