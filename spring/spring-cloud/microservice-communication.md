# MicroService Communication

## Communication Types

* 동기 방식으로 통신하는 경우 HTTP 통신을 사용할 수 있다. 이 방식을 사용하면 요청을 보냈을 때 응답이 올 때 까지 기다려야 하므로 그동안 다른 작업을 수행할 수 없다.
* 비동기 방식으로는 AMQP 를 사용할 수 있다.

## RestTemplate

* REST API 요청을 위해 스프링 프레임워크에서 제공하는 클래스이다.

### 사용 방법

* 보통은 하나의 RestTemplate 객체를 만들어 각 서비스마다 빈으로 주입해 사용한다.

```java
@Configuration
public class Config {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
@Service
public class UserService {
    
    // 생성자 주입
    private final RestTemplate restTemplate;
    
    // ...
    public UserDto getUserByUserId(String userId) {
        // user 정보 조회
        // ...
    
        // order 정보 조회
        String orderUrl = "http://.../order-service/.../orders";
        ResponseEntity<List<ResponseOrder>> response = restTemplate.exchange(orderUrl, HttpMethod.GET, null,
            new ParameterizedTypeReference<List<ResponseOrder>>() {});
        
        List<ResponseOrder> orders = response.getBody();
        userDto.setOrders(orders);
        
        return userDto;
    }
}
```

## FeignClient

* REST 호출을 추상화한 Spring Cloud Netflix 라이브러리이다.
* 호출하려는 HTTP Endpoint에 대한 인터페이스를 생성하고 @FeignClient 어노테이션을 추가해 사용할 수 있다.
* 로드밸런싱 기능을 제공한다.

### 사용 방법

* 의존성을 추가하고 @EnableFeignClients 어노테이션을 설정 클래스에 추가한다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
```

```java
@EnableFeignClients
public class Config {
}
```

* 인터페이스를 생성한다. @FeignClient 어노테이션의 name 속성에는 API 호출 대상인 마이크로 서비스의 이름을 입력한다. 인터페이스 메서드는 Controller와 동일한 형태로 작성하면 된다.

```java
@FeignClient(name=order-service)
public interface OrderServiceClient {
    @GetMapping("/order-service/{userId}/orders")
    List<ResponseOrder> getOrders(@PathVariable String userId);
}
```

* 서비스 클래스에서는 인터페이스를 빈으로 입력받아 사용할 수 있다.

```java
@Service
public class UserService {
    
    // 생성자 주입
    private final OrderServiceClient orderServiceClient;
    
    // ...
    public UserDto getUserByUserId(String userId) {
        // user 정보 조회
        // ...
    
        // order 정보 조회
        List<ResponseOrder> orders = orderServiceClient.getOrders(userId);
        
        userDto.setOrders(orders);
        
        return userDto;
    }
}
```

### 예외 처

* FeignClient 사용 중에 만약 http uri가 잘못되었다면 FeignException이 발생할 것이다.
* try-catch로 예외를 처리할 수도 있지만 ErrorDecoder를 이용해 예외를 집약적으로 처리할 수 있다.

#### 사용 방법

* Errordecoder 클래스를 생성해 각 예외 원인마다 원하는 예외 처리를 해주고 이 클래스를 빈으로 등록해주면 된다.

```java
public class FeignErrorDecoder implements ErrorDecoder {
    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400:
                break;
            case 404:
                if (methodKey.contains("getOrders")) {
                    return new ResponseStatusException(HttpStatus.valueOf(response.status()), "User's order is empty");
                }
                break;
            default:
                return new Exception(response.reason());
        }
    }
}
```

```java
@Configuration
public class Config {

    @Bean
    public FeignErrorDecoder feignErrorDecoder() {
        return new FeignErrorDecoder();
    }
}
```

