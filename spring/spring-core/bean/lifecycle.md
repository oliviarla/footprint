# Lifecycle

## 빈 생명주기

데이터베이스 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, **객체의 초기화와 종료 작업이 필요**하다.

#### **스프링 빈의 이벤트 라이프사이클**

```java
스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화콜백 -> 사용 -> 소멸전 콜백 -> 스프링 종료
```

* 일반적으로 **객체 생성** 이후 **의존관계 주입**이 일어난다.
*   초기화 작업은 의존관계 주입이 모두 완료되고 난 다음에 호출

    > **초기화 콜백**: 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
* **스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공**
*   **스프링 컨테이너가 종료되기 직전에 소멸 콜백**

    > **소멸전 콜백**: 빈이 소멸되기 직전에 호출

#### **객체의 생성과 초기화를 분리**

* 객체의 생성자는 필수 정보(파라미터)를 입력받아 필드에 저장하고, 메모리를 할당해서 객체를 생성하는 책임을 가진다.
* 실제 동작하는 행위는 별도의 초기화 메소드로 분리한다.
* 단, 초기화 작업이 내부 값들만 약간 변경하는 정도로 단순한 경우에는, 생성자에서 한번에 다 처리해도 무방하다.

## 스프링의 빈 생명주기 콜백 지원 방식

### 인터페이스 구현

* InitializingBean, DisposableBean 인터페이스를 구현하는 방식
  * `afterPropertiesSet()` : InitializingBean 상속 시 구현 필요, 객체가 생성되고 의존관계 주입이 끝난 후에 호출되는 메소드, 초기화하는 로직을 이 부분에 구현
  * `destroy()` : DisposableBean 상속 시 구현 필요, 빈이 종료될 때 호출되는 메소드, 소멸 전 콜백 이후 필요한 로직을 이 부분에 구현
* 코드가 스프링 전용 인터페이스에 의존하게 된다.
* 초기화, 소멸 메서드의 이름을 변경할 수 없다.
* 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.
* 지금은 거의 사용되지 않는다.

### @Bean에 초기화 메서드, 종료 메서드 지정

`@Bean(initMethod = "init", destroyMethod = "close")`

* 메서드 이름을 자유롭게 사용 가능
* 스프링 빈이 스프링 코드에 의존하지 않는다.
* 코드가 아니라 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다.
* `destroyMethod` 를 따로 지정하지 않으면 **(inferred)** 라는 기본값이 사용되어 close , shutdown 같은 이름을 가진 종료 메서드를 추론해서 자동으로 호출한다. 추론 기능을 사용하기 싫으면 `destroyMethod=""` 처럼 빈 공백을 지정한다.

### @PostConstruct, @PreDestroy

* 원하는 대로 구현한 메서드에 어노테이션 하나만 붙이면 되므로 자유도가 높다.
* `javax.annotation` 패키지에 속해있어 스프링에 종속적인 기술이 아니라 JSR-250 라는 자바 표준이다. 따라서 스프링이 아닌 다른 컨테이너에서도 동작한다.
* 컴포넌트 스캔과 잘 어울린다.
* 외부 라이브러리에는 적용하지 못하므로 외부 라이브러리를 초기화, 종료 해야 하면 @Bean의 메소드 지정 기능을 사용해야 한다.
* 가장 권장되는 방식이다.
  * We recommend that you do not use the `InitializingBean` interface, because it unnecessarily couples the code to Spring. Alternatively, we suggest using the [`@PostConstruct`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/postconstruct-and-predestroy-annotations.html) annotation or specifying a POJO initialization method. In the case of XML-based configuration metadata, you can use the `init-method` attribute to specify the name of the method that has a void no-argument signature. With Java configuration, you can use the `initMethod` attribute of `@Bean`. See [Receiving Lifecycle Callbacks](https://docs.spring.io/spring-framework/reference/core/beans/java/bean-annotation.html#beans-java-lifecycle-callbacks).
  * [https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-initializingbean](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle-initializingbean)
