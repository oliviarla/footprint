# Spring WebFlux와 통합

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption><p><a href="https://kouzie.github.io/spring-reative/Spring-React-Security/">https://kouzie.github.io/spring-reative/Spring-React-Security/#</a></p></figcaption></figure>

## Authentication

### AuthenticationWebFilter (인증 필터, 구현 필요)

* `setRequiresAuthenticationMatcher(ServerWebExchangeMatcher)`
  * 어떤 요청에 한해 이 필터를 활성화시킬 지 지정한다.
* `setAuthenticationConverter(Function<ServerWebExchange,Mono<Authentication>> authenticationConverter)`
  * <mark style="color:green;">`ServerWebExchange`</mark>에 담긴 Http 요청 정보를 이용해 <mark style="color:blue;">`Authentication`</mark> 객체를 만드는 함수를 입력한다.
* `ReactiveAuthenticationManager`에 의해 인증 작업이 완료되면, `ServerAuthenticationSuccessHandler` 혹은 `ServerAuthenticationFailureHandler`가 호출된다.
* 인증 작업이 성공한다면 `ReactiveSecurityContextHolder`에 <mark style="color:blue;">`Authentication`</mark> 객체가 등록된다.

### ReactiveAuthenticationManager

* 입력된 Authentication이 유효한지 검증한다.
* username, password가 일치하는지 확인하게 된다.
* `UserDetailsRepositoryReactiveAuthenticationManager` 구현체와 `ReactiveUserDetailsService` 클래스를 빈으로 등록하면 자동으로 검증 작업을 수행해준다.

### ReactiveUserDetailsService (구현 필요)

* 로그인 시에 사용자가 데이터 저장소에 있는지 검증한 후 해당 사용자 정보를 반환하는 클래스
* 직접 데이터 저장소에서 사용자 정보를 조회하는 코드를 구현해야 한다.

## Authorization

### ServerSecurityContextRepository (인가 필터의 컴포넌트, 구현 필요)

* 요청 간에 SecurityContext를 저장하기 위한 인터페이스이다.
* SecurityContextPersistenceFilter는 이 Repository를 이용해 현재 실행중인 스레드가 사용해야 할 Context를 얻는다.
* JWT 방식을 사용하는 경우 여기에서 load 메서드를 구현하면 된다. HTTP Header의 토큰을 파싱해 UsernamePasswordAuthenticationToken 객체를 만들고, SecurityContextImpl 객체에 감싸 반환하면 된다.
* Spring Session을 사용하는 경우 별도 웹 필터를 만들 필요 없이 여기에서 load 메서드를 구현하면 된다.

> WebSessionServerSecurityContextRepository 사용하면, 필요없는 SecurityContext를 추가하게 되므로 커스텀하게 구현하는 것이 낫다.

### ReactorContextWebFilter

* Spring MVC + Security의 SecurityContextPersistenceFilter와 유사하다.
* ServerSecurityContextRepository를 사용해 SecurityContext를 ReactiveSecurityContextHolder에 넣는 역할을 한다.

```java
private Context withSecurityContext(Context mainContext, ServerWebExchange exchange) {
  return mainContext.putAll(((Context)this.repository.load(exchange).as(ReactiveSecurityContextHolder::withSecurityContext)).readOnly());
}
```

### DefaultWebSessionManager

* Redis Session Repository에 실질적으로 세션 정보에 대한 요청을 보내는 클래스
* 사용자로부터 요청이 들어와 DefaultServerWebExchange 생성 시에 sessionMono가 생성되며 나중에 필요 시 이 DefaultWebSessionManager가 사용된다.

```java
this.sessionMono = sessionManager.getSession(this).cache();
```

* 만약 HTTP 쿠키에 세션이 없는 경우 기본 세션을 생성하게 되며, 이 때에는 attributes가 없는 상태가 된다.

### SpringSessionWebSessionStore

* SpringSessionWebSession을 만들고 관리하는 클래스
* Spring Session Redis를 사용하는 경우 RedisSession을 SpringSessionWebSession에 감싸 사용하게 된다.
* session attributes는 SpringSessionMap 형태로 관리된다.

### ReactiveSecurityContextHolder

* Reactor Context 객체에 저장된 SecurityContext를 관리한다.
* `getContext()` 메서드로 SecurityContext 객체를 얻어 Authentication 객체에 담긴 정보를 사용할 수 있다.

## Logout

### LogoutHandler (구현 필요)

* Spring Session과 통합하여 사용하거나 JWT를 사용할 때, 기본 LogoutHandler인 `SecurityContextServerLogoutHandler`가 session.invalidate()를 호출하거나 토큰을 제거해주지 않으므로, 직접 LogoutHandler를 구현해 호출해야 한다.

### LogoutWebFilter

* 로그아웃을 담당하는 필터 클래스
* LogoutHandler를 사용해 logout 과정을 진행하고, 성공 시 SuccessHandler를 사용한다.
* 로그아웃 완료 시 ReactiveSecurityContextHolder의 Context를 초기화해준다.
