# Service Discovery

## 개념

* 외부에서 마이크로서비스를 검색할 때 사용하는 컴포넌트
* 마이크로 서비스를 Service Discovery에 먼저 등록하고, Service Discovery는 클라이언트의 요청을 처리할 수 있는 마이크로서비스에 이를 전달한다.

## Spring Cloud Netflix Eureka

* Service Discovery의 역할을 할 수 있도록 제공하는 라이브러리다.
* Eureka에 서비스를 등록하려면 아래와 같은 설정 파일을 둘 수 있다.
  * register-with-eureka: Eureka 서버에 현재 서비스를 등록할 지 설정한다.
  * fetch-registry: Eureka 서버로부터 인스턴스의 정보들을 주기적으로 가져올 지 설정한다.
  * service-url: 구동중인 Eureka 서버의 정보를 입력한다.

```yaml
server:
    port: 9001
spring:
    application:
        name: user-service
eureka:
    client:
        register-with-eureka: true
        fetch-registry: true
        service-url:
            defaultZone: http://127.0.0.1:8761/eureka
```

* Eureka에 서비스를 등록하기 위해 아래와 같이 EnableDiscoveryClient 어노테이션을 붙여주어야 한다.

```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

### 랜덤 포트

* 랜덤 포트 사용 시 유레카 대시보드에서 0이라는 숫자가 그대로 나타나기 때문에 instance-id를 지정하여 대시보드에서 정보가 보이도록 해야 한다.

```yaml
server:
    port: 0
eureka:
    client:
        register-with-eureka: true
        fetch-registry: true
        service-url:
            defaultZone: http://127.0.0.1:8761/eureka
    instance:
        instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```
