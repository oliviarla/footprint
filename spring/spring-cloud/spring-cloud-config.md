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

* 로컬 폴더의 파일들을 적용하고자 한다면 아래와 같이 설정하면 된다.

```yaml
spring:
    application:
        name: config-service
    cloud:
        config:
            server:
                native:
                    search-locations: file:///Users/...
```

* Spring Cloud Config Server는 설정 파일에 명시된 파일 혹은 링크 정보를 실시간으로 반영한다.
* 반영된 설정 파일을 조회하려면 서버 주소에 `/<config-name>/<profile>` 요청을 보내면 된다.

## Spring Cloud Config Client

* Spring Cloud Config로부터 정보를 받아오기 위해 마이크로 서비스 프로젝트에 아래와 같이 의존성을 추가하고 bootstrap.yml 파일을 추가해야 한다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-config'
implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'
```

* 만약 Spring Cloud Config Server의 application name을 spring.cloud.config.name 속성에 지정한다면 application.yml 파일이 사용된다.
* 아래와 같이 ecommerce 같은 별도의 이름을 지정한다면 ecommerce.yml 파일이 사용될 것이다.

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
* Spring Cloud Bus는 분산 시스템의 노드를 경량 메시지 브로커와 연결한다. 그리고 상태 및 구성에 대한 변경 사항을 메시지 브로커를 통해 연결된 노드들에게 전달하는 **Broadcast 방식을 사용**한다.
* 특정 마이크로 서비스에 대해 `POST /busrefresh` http 요청을 보내면 Spring Cloud Bus에 의해 다른 모든 마이크로 서비스들에 대해 Config 정보가 갱신된다.

#### 메시지 브로커

* AMQP(Advanced Message Queuing Protocol)
  * 메시지 지향 미들웨어를 위한 개방형 표준 응용 계층 프로토콜이다.
  * 메시지 지향, 큐잉, 라우팅(P2P / Pub-Sub) 지원, 신뢰성, 보안 등의 특징을 가진다.
  * Erlang, RabbitMQ에서 사용한다.
* RabbitMQ
  * 메시지 브로커 프로그램이다.
  * 초당 20개 이상의 메시지를 Subscriber에게 전달한다.
  * 메시지 전달을 보장하며 시스템 간 메시지를 전달한다.
  * 브로커, Subscriber 중심이다.
* Kafka
  * Apache 재단이 Scalar 언어로 개발한 오픈소스 메시지 브로커이다.
  * 분산형 스트리밍 플랫폼으로 대용량의 데이터를 처리할 수 있는 메시징 시스템이다.
  * 초당 100k 이상의 이벤트를 처리할 수 있다.
  * Pub/Sub, Topic 방식으로 데이터를 주고받는다.
  * Ack을 기다리지 않고 메시지 전달이 가능하다.
  * Publisher 중심이다.

#### 사용 방법

* Spring Cloud Config Server에 아래와 같이 의존성을 추가하고 설정 파일을 수정한다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'
```

```yaml
spring:
    application: # ...
    rabbitmq:
        host: 127.0.0.1
        port: 5672
        username: guest
        password: guest
management:
    endpoints:
        web:
            exposure:
                include: busrefresh
```

* Spring Cloud Config Client(마이크로 서비스들, Gateway Service)에 아래와 같이 의존성을 추가하고 설정 파일을 수정한다. Gateway를 비롯한 마이크로 서비스들은 RabbitMQ의 데이터를 받아오는 Subscriber가 된다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
```

```yaml
spring:
    application: # ...
    rabbitmq:
        host: 127.0.0.1
        port: 5672
        username: guest
        password: guest
management:
    endpoints:
        web:
            exposure:
                include: busrefresh
```

* 이제 RabbitMQ에 연결된 여러 Spring Cloud Config Client 중 하나에 `POST /busrefresh` http 요청을 보내면, Spring Cloud Bus에 의해 다른 모든 마이크로 서비스들에도 Config 정보가 갱신된다.

## Configuration 암호화

### 개요

* 암호화(Encryption), 복호화(Decryption) 과정을 거치며 Configuration을 안전하게 보관하고 사용하는 방법을 알아본다.
* 전혀 알수 없는 데이터 타입으로 Configration을 변형해 저장하고, 사용 시점에 이를 복호화해서 사용하도록 한다.

### 대칭키를 이용한 암호화

* 암호화/복호화 시에 같은 키를 사용하는 방식이다.
* 마이크로 서비스의 application.yml 파일에 중요한 비밀번호 등의 정보를 담고 있으면 유출의 위험이 있다.
* 이러한 문제를 방지하기 위해 기존에 application.yml 파일에 있던 민감한 정보들을 Config Server에서 관리할 수 있도록 `xxx-service.yml` 파일로 이동시킨다. 그리고 비밀번호와 같은 데이터를 암호화해 저장해둔다.
  * `{cipher}` 라고 명시해주어야 암호화되었는지 여부를 Bootstrap에서 확인하여 복호화를 해줄 수 있다.

```yaml
spring:
    datasource:
        driver-class-name: org.h2.driver
        url: jdbc:h2:mem:testdb
        username: sa
        password: '{cipher}abcd...'
```

* Config Server에서는 Spring Cloud Starter Bootstrap 라이브러리를 통해 암호화되어 있던 필드를 복호화하기 위해 bootstrap.yml 파일에 대칭키를 적어준다.

```yaml
encrpyt:
    key: abcedfg1234...
```

* Spring Cloud Config Server에 `POST /encrypt` , `POST /decrypt` http 요청을 데이터를 넣어 보내면 각각 암호화/복호화된 결과를 반환해준다.
* 그리고 마이크로 서비스 프로젝트의 bootstrap.yml 파일에 Spring Cloud Config로부터 사용할 설정 파일의 이름을 입력해준다.

```yaml
spring:
    cloud:
        config:
            uri: # Spring Cloud Config Server 주소
            name: xxx-service
```

### 비대칭키를 이용한 암호화

* 암호화/복호화 시에 다른 키를 사용한다.
* public, private 키를 두고 사용하게 된다.
* 복호화할 때에는 암호화할 때 사용하지 않은 키를 써야 한다.
* RSA Keypair 방식에서 사용된다.
* Java에서 제공하는 keytool을 사용해 public key, private key를 생성할 수 있다.
* 아래와 같이 keytool을 이용해 키 파일을 만든다.
  * \-alias : 키 별칭
  * \-keyalg : 키 생성 알고리즘
  * \-keypass : 키 생성을 위한 비밀번호
  * \-keystore : 키 생성을 저장할 곳
  * \-store : 저장되는 파일을 위한 비밀번호

{% code overflow="wrap" %}
```bash
$ keytool -genkeypair -alias keyset -keyalg RSA -dname "CN=Kim, OU=API Development, O=abc.co.kr, L=Seoul, C=KR" -keypass "1q2w3e4r" -keystore keyset.jks -storepass "1q2w3e4r"
```
{% endcode %}

* 아래 명령을 통해 키 정보를 확인할 수 있다.

```bash
$ keytool -list -keystore keyset.jks -v
```

* 아래 명령을 통해 인증서 파일을 생성하고, 인증서 파일을 다시 키 파일로 변경할 수 있다.

```bash
$ keytool -export -alias keyset -keystore keyset.jks -rfc -file trustServer.cer

$ keytool -import -alias trustServer -file trustServer.cer -keystore publicKey.jks
```

* keytool으로 생성한 키 파일에 대한 정보를 Config Server에 명시해주면 해당 키 파일을 이용해 암호화/복호화를 진행하게 된다.

```yaml
encrypt:
    key-store:
        location:file://...
        password: 1q2w3e4r
        alias: keyset
```

* Config Server에서 관리할 수 있는 `xxx-service.yml` 파일에 비밀번호와 같은 데이터를 암호화해 저장해둔다.
  * 암호화된 데이터에는 앞에 `{cipher}` 라고 명시해주어야 암호화되었는지 여부를 Bootstrap에서 확인하여 복호화를 해줄 수 있다.

```yaml
spring:
    datasource:
        driver-class-name: org.h2.driver
        url: jdbc:h2:mem:testdb
        username: sa
        password: '{cipher}abcd...'
```
