# Microservice 간 분산 추적

## 구성

### Zipkin

* 여러 서비스가 연결된 상태에서 시스템에 병목 현상이 발생했을 때 어떤 부분에서 문제가 발생했는지 파악하기 위한 목적으로 사용한다.
* 외부로부터 트레이스 정보를 수집하여 저장소에 저장하고, 트레이스 정보를 시각화하여 제공한다.
* TraceId를 사용해 어떤 서비스들로 요청 흐름이 들어갔는지 확인할 수 있다.
* **Span**
  * 하나의 요청 내에서 수행되는 하위 작업의 단위
* **Trace**
  * 트리 구조로 이뤄진 Span들의 집합
  * 사용자의 요청에 따라 하나의 `TraceId`가 생성되고, 각 마이크로 서비스에 대한 요청을 수행할 때 `SpanId`가 생성된다.
* Span, Trace에 대한 정보를 B3 Propagation을 통해 관리한다.
  * Http Request header 또는 Kafka header를 통해 TraceId, ParentSpanId, SpanId, Sampled 데이터를 전달한다.

### ~~Spring Cloud Sleuth~~ -> Micrometer

* 로그 파일 TraceId, SpanId를 Zipkin 서버에 전달하여 데이터를 Zipkin이 처리할 수 있도록 한다.
* Spring 3.1.X 버전에 들어서면서 트레이싱 기능이 Micrometer로 이전되었다. 이를 통해 Spring Cloud 뿐만 아니라 Spring Framework, Spring Boot 레벨에서 트레이싱 기능을 사용할 수 있게 되었다.

## 적용

* 아래와 같은 의존성을 추가하면 서비스에서 Zipkin을 연동할 수 있다.

```groovy
implementation 'io.micrometer:micrometer-observation'
implementation 'io.micrometer:micrometer-tracing-bridge-brave'
implementation 'io.zipkin.brave:brave-instrumentation-spring-web'
implementation 'io.zipkin.reporter2:zipkin-reporter-brave'
```

**출처**

* [https://engineering.linecorp.com/ko/blog/line-ads-msa-opentracing-zipkin](https://engineering.linecorp.com/ko/blog/line-ads-msa-opentracing-zipkin)
* [https://techblog.lycorp.co.jp/ko/how-to-migrate-to-spring-boot-3](https://techblog.lycorp.co.jp/ko/how-to-migrate-to-spring-boot-3)
