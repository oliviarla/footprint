# 어노테이션 권한 설정 방식

## @PreAuthorize

* SpEL을 지원한다.
* 미리 권한을 검사하여 적합한 사용자라면 메서드에 진입할 수 있다.

```java
@PreAuthorize("hasRole('ROLE_USER’) and (#account.username == principal.username)")
```

PrePostAnnotationSecurityMetadataSource 클래스가 이 부분을 직접 처리한다.

## @PostAuthorize

*

## @Secured

* SpEL을 지원하지 않는다.
* 단순한 문자열로 권한이 있는지 확인할 수 있다.
* SecuredAnnotationSecurityMetadataSource 클래스가 이부분을 담당한다.

## @RolesAllowed

*
* 단순한 문자열로 권한이 있는지 확인할 수 있다.
* Jsr250MethodSecurityMetadataSource 클래스가 이 부분을 담당한다.

## @EnableGlobalMethodSecurity

* Configuration 클래스에 이 어노테이션을 설정해주어야 정상적으로 어노테이션 방식을 사용할 수 있다.

```java
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfiguration { ... }
```



