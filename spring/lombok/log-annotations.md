# Log Annotations

* Lombok에서는 다양한 Logging 관련 어노테이션을 제공한다.
* Java에서 제공해주는 기본 Logger를 비롯하여 Log4j, Slf4j, CommonsLog, Flogger, JBossLog에 대해 어노테이션만 선언하면 Logger 객체를 사용할 수 있도록 해준다.

## @Slf4j

* 어노테이션 기반으로 직접 Logger 객체를 만들지 않고도 log 객체를 사용해 로깅할 수 있도록 한다.
* 클래스와 Enum 타입에 적용할 수 있다.

```java
 @Slf4j
 public class LogExample {
     public void example() {
        log.info("example");
    }
 }
```

* 컴파일 과정에서는 아래와 같이 Logger 객체를 생성해준다.

```java
 public class LogExample {
     private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
 }
```

#### 출처

[https://projectlombok.org/features/log](https://projectlombok.org/features/log)
