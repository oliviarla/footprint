# 어노테이션 권한 설정 방식

## @PreAuthorize

* SpEL을 지원한다.
* 미리 권한을 검사하여 적합한 사용자에 한해 메서드 내부로 진입할 수 있다.

```java
@PreAuthorize("hasRole('ROLE_USER’) and (#account.username == principal.username)")
```

PrePostAnnotationSecurityMetadataSource 클래스가 이 부분을 직접 처리한다.

## @PostAuthorize

* 메서드 실행 후 클라이언트에게 응답하기 전 권한을 검사하여 응답 여부를 판단한다.
* 메서드 결과값을 확인해 특정 조건을 만족해야만 응답을 하도록 할 수도 있다.

## @Secured

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

## @RolesAllowed

* 자바 표준에서 지원하는 어노테이션이다.
* 단순한 문자열로 권한이 있는지 확인할 수 있다.
* @Secured와 사용법은 같다.
* Jsr250MethodSecurityMetadataSource 클래스가 이 부분을 담당한다.
