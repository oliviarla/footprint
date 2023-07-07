# 인증 아키텍처

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

### 동작 흐름

> 한번 보면 쉽게 이해되지 않겠지만 여러 번 보다보면 익숙해진다 :joy:

1. 사용자가 로그인 API 요청을 보내면 <mark style="color:blue;">`AuthenticationFilter`</mark>로 요청 정보가 들어간다.
2. <mark style="color:blue;">`AuthenticationFilter`</mark>로 로그인 요청이 들어오면, 아이디와 비밀번호를 기반으로 <mark style="color:orange;">`UsernamePasswordAuthenticationToken`</mark> (이하 <mark style="color:orange;">`Token`</mark>)를 생성한다.
   * `CustomAuthenticationFilter`
     * <mark style="color:blue;">`AuthenticationFilter`</mark> 의 역할을 하는 클래스를 우리가 직접 구현해야 인증을 진행할 수 있다.&#x20;
     * 보통 UsernamePasswordAuthenticationFilter 필터를 상속받는다.
   * &#x20;<mark style="color:orange;">`Token`</mark> 은 `Authentication` 인터페이스를 구현한 객체이다.
3. <mark style="color:orange;">`Token`</mark>을 <mark style="color:red;">`AuthenticationManager`</mark>로 전달한다.
4. `AuthenticationManager`는 전달받은 <mark style="color:orange;">`Token`</mark>을 <mark style="color:green;">`AuthenticationProvider`</mark>들로 전달해 실제 인증 과정을 시작한다. <mark style="color:orange;">`Token`</mark>으로부터 아이디를 조회해 `UserDetailsService`로 넘긴다.
5. `UserDetailsService` 에서는 아이디를 입력받아 DB에서 유저 데이터를 조회하고, 결과를 UserDetails 객체로 반환한다.
   * `CustomUserDetailsService`
     * `UserDetailsService` 를 구현하는 클래스
     * loadUserByUsername() 메서드를 구현한 후 `AuthenticationManager` 에 등록해야 한다
6. \-
7. `UserDetailsService` 에서 아이디 기반으로 조회된 `UserDetails` 결과는 <mark style="color:green;">`AuthenticationProvider`</mark>로 반환된다.
8. <mark style="color:green;">`AuthenticationProvider`</mark> 에서는 `UserDetails` 객체와  토큰으로부터 입력받은 정보를 비교해 인증이 완료되면 <mark style="color:orange;">`Token`</mark> 객체를 다시 생성해 <mark style="color:red;">`AuthenticationManager`</mark>로 반환한다.
9. <mark style="color:blue;">`AuthenticationFilter`</mark> 는 인증된 <mark style="color:orange;">`Token`</mark>을 받게 되면 `LoginSuccessHandler` 로 이동한다.
   * `CustomLoginSuccessHandler`
     * `CustomAuthenticationFilter` 에서 인증이 성공적으로 수행된 후에 처리될 Handler 클래스
       * <mark style="color:green;">`AuthenticationProvider`</mark>를 통해 인증이 성공될 경우 처리된다.
       * Bean으로 등록하고 SecurityConfig에서 `CustomAuthenticationFilter`의 핸들러로 추가해준다.
       * 로그인 성공 시 JWT 토큰을 생성하고, `HttpServletResponse` 에 이를 헤더 또는 쿠키에 추가해 HTTP 응답을 보낸다.
10. `LoginSuccessHandler` 에서는 <mark style="color:orange;">`Token`</mark> 객체를 스프링 시큐리티가 관리하는 SecurityContext에 저장한다.

