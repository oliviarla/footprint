# 메소드별 인가 처리

## 개요

* 스프링 시큐리티에서는 특정 권한을 가진 사용자나, 로그인 중인 사용자에 한해 API 요청을 받아들이고 메서드를 수행할 수 있도록 제한하는 기능을 제공한다. 즉, 기능(메서드) 단위로 인가를 처리할 수 있다는 것이다.
* 사용자는 다양한 방식을 사용해 어떤 사용자에게만 API를 허용할 지 지정할 수 있다.
* 어노테이션 기반으로 권한을 설정하면 AOP 기반으로 프록시와 어드바이스를 통해 메소드 인가 처리가 수행된다.

## 사용법

* 어노테이션 기반 권한 검증을 수행하려면 Configuration 클래스에 `@EnableMethodSecurity` 어노테이션을 붙여야 한다.
* `@EnableMethodSecurity` 어노테이션에서는 어느 권한 검증 방식을 활성화할 지 정할 수 있다.
  * `prePostEnabled` : `@PreAuthorize` , `@PostAuthorize` 사용 시 활성화되어야 하며, 기본 값이 true이므로 굳이 설정할 필요는 없다.
  * `securedEnabled` : `@Secured` 어노테이션 사용 시 true로 설정해야 한다.
  * `jsr250Enabled` : `@RoleAllowed` 어노테이션 사용 시 true로 설정해야 한다.

> Spring 5.6 이전이라면 `@EnableGlobalMethodSecurity` 어노테이션을 붙여야 한다. 두 어노테이션 간 기본값이 다르니 주의하여야 한다.

* 아래와 같이 Configuration 클래스에 어노테이션을 추가해주면 된다.

```java
// ...
@EnableMethodSecurity
public class SecurityConfig {
  // ...
}
```

* 이후에는 아래와 같이 어노테이션을 사용해 Controller에서 조건에 맞는 사용자에 한해 API를 정상 수행할 수 있다.

```java
@GetMapping("/users")
@PreAuthorize("isAuthenticated() && hasRole('ADMIN')")
public List<String> getUsers(@AuthenticationPrincipal User user) {
	return userService.getAllSessionsByUser(user.getUsername());
}
```

## 종류

### @PreAuthorize

* SpEL을 지원한다.
* 미리 권한을 검사하여 적합한 사용자에 한해 메서드 내부로 진입할 수 있다.

```java
@PreAuthorize("hasRole('ROLE_USER’) and (#account.username == principal.username)")
```

PrePostAnnotationSecurityMetadataSource 클래스가 이 부분을 직접 처리한다.

### @PostAuthorize

* 메서드 실행 후 클라이언트에게 응답하기 전 권한을 검사하여 응답 여부를 판단한다.
* 메서드 결과값을 확인해 특정 조건을 만족해야만 응답을 하도록 할 수도 있다.

### @Secured

* Spring Security에서 지원하는 어노테이션이다.
* SecuredAnnotationSecurityMetadataSource 클래스가 이부분을 담당한다.
*   SpEL은 지원하지 않으며, 아래와 같이 단순한 문자열로 권한이 있는지 확인할 수 있다.

    ```java
    @Secured({"ROLE_USER", "ROLE_ADMIN"})
    public User me() { ... }
    ```
*   Configuration 클래스에 아래 어노테이션을 설정해주어야 정상적으로 동작한다.

    ```java
    @EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
    public class SecurityConfiguration { ... }
    ```

### @RolesAllowed

* 자바 표준에서 지원하는 어노테이션이다.
* 단순한 문자열로 권한이 있는지 확인할 수 있다.
* @Secured와 사용법은 같다.
* Jsr250MethodSecurityMetadataSource 클래스가 이 부분을 담당한다.
