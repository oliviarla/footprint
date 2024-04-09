# Spring Cloud Netflix Eureka

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
