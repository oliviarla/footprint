# 빌더 패턴

## 접근

* 많은 필드와 중첩된 객체들을 단계별로 초기화해야 할 때 많은 매개 변수를 포함한 거대 생성자를 사용하면 코드가 복잡해진다. 반드시 필요하지 않은 값들도 매개 변수로 정의되므로 null을 입력해야만 하는데, 이러면 가독성이 안좋아진다.
* 객체 생성 코드를 추출해 빌더 클래스에 단계 별로 구성 요소를 생성하도록 해야 한다.

## 개념

* 클라이언트가 객체를 생성할 때 다양한 방법으로 생성할 수 있도록 빌더 클래스를 제공하는 패턴이다.
* 복합 객체 구조를 생성할 때 많이 쓰인다.
* 아래와 같이 Builder 인터페이스를 구현하여 다양한 생성 단계를 가진 빌더 객체를 만들 수 있다. 각 빌더 객체로부터 얻은 결과 객체는 같은 클래스 계층구조 또는 인터페이스에 속할 필요가 없다. 디렉터 클래스를 생성하면 빌더 객체로 객체를 생성하는 방법을 캡슐화할 수 있다.

<figure><img src="../../../.gitbook/assets/image (2).png" alt="" width="360"><figcaption></figcaption></figure>

* 아래와 같이 점층적 생성자로 인해 코드가 복잡해질 경우 빌더 패턴을 사용해 클라이언트가 필요한 단계들만 설정해 객체를 생성하도록 하면 된다.

```java
class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // …
```

```java
PizzaBuilder pb = new PizzaBuilder();
pb.size("LARGE");
pb.crust("Cheese");
pb.topping("pineapple", "olive");
Pizza pizza = pb.getPizza();
```

* 빌더의 생성 단계들을 재귀적으로 동작하도록 하여 컴포지트 패턴의 트리를 생성할 수 있다.

## 장단점

* 장점
  * 복합 객체 생성 과정을 캡슐화한다.
  * 팩토리 패턴과 달리 여러 단계와 다양한 절차를 거쳐 객체를 만들 수 있다.
  * 클라이언트는 추상 인터페이스만 볼 수 있으므로 실제 빌더 구현체를 쉽게 바꿀 수 있다.
* 단점
  * 팩토리를 사용할 때 보다 클라이언트에 관해 많이 알아야 한다.

## 사용 방법

* 객체의 공통 생성 단계를 정의하고 빌더 인터페이스에서 이를 선언한다.
* 빌더 인터페이스를 구현하는 클래스를 만든다. 필요하다면 디렉터 클래스도 만든다.
* 클라이언트 코드에서 빌더 객체나 디렉터 객체를 사용해 원하는 객체를 생성한다.

## 예시

* 아래에서 소개할 두 예시는 사실 완벽한 GoF 빌더 패턴은 아니다. 보통 GoF에서 제시한 것 처럼 빌더 인터페이스와 구현체를 따로 두지 않고 **간편하게 정적 내부 클래스로 빌더를 정의**한다.

### Spring Cacheable의 빌더

* 빌더 클래스를 정적 내부 클래스로 선언하고, 빌더 객체를 얻기 위해 builder 정적 메서드를 제공한다.
* 다양한 초기화 메서드를 사용해 원하는 값들을 설정하고 build 메서드를 호출하면 생성된 객체를 얻을 수 있다.

```java
public static RedisCacheManagerBuilder builder(RedisCacheWriter cacheWriter) {

  Assert.notNull(cacheWriter, "CacheWriter must not be null");

  return RedisCacheManagerBuilder.fromCacheWriter(cacheWriter);
}

public static class RedisCacheManagerBuilder {
  public static RedisCacheManagerBuilder fromCacheWriter(RedisCacheWriter cacheWriter) {

    Assert.notNull(cacheWriter, "CacheWriter must not be null");

    return new RedisCacheManagerBuilder(cacheWriter);
  }
  
  // ...
  private final Map<String, RedisCacheConfiguration> initialCaches = new LinkedHashMap<>();

  private RedisCacheConfiguration defaultCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();

  private @Nullable RedisCacheWriter cacheWriter;

  private RedisCacheManagerBuilder(RedisCacheWriter cacheWriter) {
    this.cacheWriter = cacheWriter;
  }

  public RedisCacheManagerBuilder allowCreateOnMissingCache(boolean allowRuntimeCacheCreation) {
    this.allowRuntimeCacheCreation = allowRuntimeCacheCreation;
    return this;
  }
  // ...

  public RedisCacheManager build() {

    Assert.state(cacheWriter != null, "CacheWriter must not be null;"
        + " You can provide one via 'RedisCacheManagerBuilder#cacheWriter(RedisCacheWriter)'");

    RedisCacheWriter resolvedCacheWriter = !CacheStatisticsCollector.none().equals(this.statisticsCollector)
        ? this.cacheWriter.withStatisticsCollector(this.statisticsCollector)
        : this.cacheWriter;

    RedisCacheManager cacheManager = newRedisCacheManager(resolvedCacheWriter);

    cacheManager.setTransactionAware(this.enableTransactions);

    return cacheManager;
  }
}
```

### 롬복의 @Builder

* 아래와 같이 필요한 인자를 입력받는 생성자에 `@Builder` 어노테이션을 붙이면 컴파일 시 롬복이 자동으로 빌더 클래스를 추가해준다.

```java
@Builder
public User(String username, String password, String email, String phoneNumber, Role role) {
  this.username = username;
  this.password = password;
  this.email = email;
  this.phoneNumber = phoneNumber;
  this.role = role;
}
```

* 아래는 컴파일 후 class 파일을 디컴파일해본 결과이며, 자동으로 UserBuilder 클래스가 생성된 것을 확인할 수 있다.

```java
public class User {
  // ...
  
  @Generated
  public static UserBuilder builder() {
    return new UserBuilder();
  }

  @Generated
  public static class UserBuilder {
    @Generated
    private String username;
    @Generated
    private String password;
    @Generated
    private String email;
    @Generated
    private String phoneNumber;
    @Generated
    private Role role;
  
    @Generated
    UserBuilder() {
    }
  
    @Generated
    public UserBuilder username(final String username) {
      this.username = username;
      return this;
    }
  
    @Generated
    public UserBuilder password(final String password) {
      this.password = password;
      return this;
    }
  
    @Generated
    public UserBuilder email(final String email) {
      this.email = email;
      return this;
    }
  
    @Generated
    public UserBuilder phoneNumber(final String phoneNumber) {
      this.phoneNumber = phoneNumber;
      return this;
    }
  
    @Generated
    public UserBuilder role(final Role role) {
      this.role = role;
      return this;
    }
  
    @Generated
    public User build() {
      return new User(this.username, this.password, this.email, this.phoneNumber, this.role);
    }
  
    @Generated
    public String toString() {
      String var10000 = this.username;
      return "User.UserBuilder(username=" + var10000 + ", password=" + this.password + ", email=" + this.email + ", phoneNumber=" + this.phoneNumber + ", role=" + String.valueOf(this.role) + ")";
    }
  }
}
```
