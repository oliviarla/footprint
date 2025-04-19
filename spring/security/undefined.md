# 로그아웃 처리

## 로그아웃 처리 방법

* [Spring Security 공식 문서](https://docs.spring.io/spring-security/reference/servlet/authentication/logout.html#creating-custom-logout-endpoint)에 의하면, Logout을 위한 엔드포인트, 즉 Controller에 Logout API를 작성하는 것보다는 **LogoutFilter를 사용하는 것을 권장**한다.
* SecurityContextLogoutHandler 라는 클래스에서는 HttpSession을 invalidate해주거나, SecurityContext에 저장되어 있는 Authentication을 제거해주기 때문에 안전한 로그아웃 위해 사용하는 것을 권장한다.
  * Logout API를 따로 작성하면 이 핸들러를 직접 생성하거나 주입받아 아래와 같이 코드를 작성해주어야 하므로 번거로워진다.
  * `new SecurityContextLogoutHandler().doLogout(request, response, authentication);`

## LogoutFilter 사용 방법

* SecurityConfiguration의 SecurityFilterChain에서 간단하게 로그아웃 처리하는 코드를 작성할 수 있다.
* `logoutRequestMatcher` : 어떤 URL일 때 LogoutFilter가 실행될 지 지정할 수 있다.
* `addLogoutHandler` : Logout시에 어떤 동작을 할 것인지에 대해 클래스 형태 또는 람다 형태로 정의할 수 있다. 아래 예시에서는 간단하게 람다를 사용해 쿠키를 제거해주었다.
* `logoutSuccessHandler` : 로그아웃 필터에서 모든 작업이 오류 없이 수행되었을 때 어떤 동작을 할 것인지에 대해 클래스 형태 또는 람다 형태로 정의할 수 있다. 아래 예시에서는 간단하게 response status를 OK로 변경해주었다.
* 이외에도 `deleteCookies` , `clearAuthentication` 등 로그아웃 요구사항에 맞게 다양하게 커스텀하여 활용할 수 있다.

```java
httpSecurity
    // ...
    .logout(
        httpSecurityLogoutConfigurer ->
                httpSecurityLogoutConfigurer
                    .logoutRequestMatcher(new AntPathRequestMatcher("/v1/logout", "POST"))
                    .addLogoutHandler(
                        (request, response, authentication) ->
                            Arrays.stream(request.getCookies())
                                .map(
                                    cookie -> {
                                      cookie.setMaxAge(0);
                                      cookie.setValue(null);
                                      return cookie;
                                    })
                                .forEach(response::addCookie))
                    .logoutSuccessHandler(
                        (request, response, authentication) ->
                            response.setStatus(HttpServletResponse.SC_OK)))
    )
```

## LogoutFilter

* LogoutFilter를 활성화하면 기본적으로 LogoutSuccessHandler 하나와 LogoutHandler 두 개가 등록된다.
* 커스텀 LogoutSuccessHandler가 등록되면, 해당 LogoutSuccessHandler만 동작하게 된다.
* 커스텀 LogoutHandler가 등록되면, 기존 LogoutHandler와 함께 커스텀 LogoutHandler도 동작하게 된다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
