# Spring Cloud Config

* 분산 시스템에서 서버 구동에 필요한 설정 정보를 하나의 중앙화된 외부 시스템에서 관리하기 위해 제공하는 라이브러리이다.
* 각 서비스를 다시 빌드하지 않고 설정 프로퍼티를 바로 적용할 수 있다.
* 애플리케이션 배포 파이프라인 주기 (`개발 환경 - 테스트 환경 - 프로덕션 환경`)에 맞는 구성 정보도 관리해준다.

## Spring Cloud Config Server

* Config를 담아두는 서버를 독립적으로 구성하기 위해 새로운 프로젝트를 생성하고 구동해야 한다.
* Spring Cloud Config Server 의존성을 추가하고 `@EnableConfigServer` 어노테이션을 추가한다.
* 다음은 로컬에 존재하는 git 프로젝트 디렉토리를 Spring Cloud Config에 등록하기 위해 작성한 application.yml 파일이다.
* uri 속성에 github repository 주소를 입력해도 동작한다. 만약 github repository가 private이라면 username, password 속성을 같이 입력해주어야 한다.

```yaml
spring:
    application:
        name: config-service
    cloud:
        config:
            server:
                git:
                    uri: file:///Users/...
```

* Spring Cloud Config로부터 정보를 받아오기 위해 아래와 같이 의존성을 추가하고 bootstrap.yml 파일을 추가해야 한다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-config'
implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'
```

```yaml
spring:
    cloud:
        config:
            uri: http://127.0.0.1:8888 # spring cloud config server의 주소
            name: ecommerce # spring cloud config server에 존재하는 설정 파일의 이름
```

* DEV, PROD, TEST 등 profile을 바꾸어 사용하기 위해서는 Spring Cloud Config에 \<name>-dev.yml, \<name>-prod.yml과 같이 설정 파일을 따로 저장해두고, 아래와 같이 bootstrap.yml 파일에 어떤 profile을 사용할 지 지정해주면 된다.

```yaml
spring:
    cloud:
        config:
            uri: http://127.0.0.1:8888
            name: ecommerce
    profiles:
        active: dev
```

## Configuration 변경 사항 적용

* Configuration 정보가 변경되었을 때 이를 반영하기 위해서는 아래 방법 중 하나를 사용하면 된다.
  * 서버 재구동
  * Actuator Refresh
  * Spring Cloud Bus 사용
* 서버 재구동을 하는 방식은 실제 운영 환경에서는 사실 상 사용하기 어렵다. 따라서 Actuator 사용하거나 Spring Cloud Bus를 사용하는 것이 바람직하다.

### Actuator Refresh

* Actuator Refresh 방식은 POST 방식으로 `/actuator/health` 에 요청을 보내면 변경된  spring cloud config의 정보를 반영해준다.
* Actuator 사용을 위해 아래와 같이 의존성을 추가해준다. 만약 spring security에 의해 권한을 제어하고 있다면 `/actuator/**` 엔드포인트 역시 권한 정보를 설정해주어야 한다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

* Actuator에는 다양한 정보를 볼 수 있는 엔드포인트를 제공하는데, refresh 방식을 사용하려면 아래와 같이 `application.yml` 파일에 노출시킬 엔드포인트를 지정해주어야 한다.

```yaml
management:
    endpoints:
        web:
            exposure:
                include: refresh
```

### Spring Cloud Bus

* Actuator Refresh 방식은 config를 갱신하기 위해 여러 마이크로 서비스에 직접 `/actuator/refresh` 요청을 보내주어야 한다. 이는 연동된 마이크로 서비스가 많아질수록 불편해지는데, Spring Cloud Bus를 사용하면 이러한 문제를 해결할 수 있다.
* 분산 시스템의 노드를 경량 메시지 브로커와 연결한다.
* 상태 및 구성에 대한 변경 사항을 연결된 노드에게 전달하는 Broadcast 방식을 사용한다.











