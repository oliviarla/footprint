# 로그인 처리

> 한번 보면 쉽게 이해되지 않겠지만 여러 번 보다보면 익숙해진다 :joy: 화이팅!
>
> 필터 방식에 대해 잘 모른다면 Spring MVC Lifecycle에 대해 먼저 공부하고 오는 것이 좋다.

## 로그인 처리 방법

<figure><img src="../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>

그림에 적힌 번호대로 어떤 작업이 이루어지는지 확인해보자.

1. 사용자가 로그인 API 요청을 보내면 <mark style="color:blue;">`AuthenticationFilter`</mark>로 요청 정보(아이디, 비밀번호)가 주입된다.
2. <mark style="color:blue;">`AuthenticationFilter`</mark> 에서는 아이디와 비밀번호를 기반으로 <mark style="color:orange;">`UsernamePasswordAuthenticationToken`</mark> (이하 <mark style="color:orange;">`Token`</mark>)를 생성한다.
   * <mark style="color:blue;">`AuthenticationFilter`</mark> 의 역할을 하는 클래스를 우리가 직접 구현해야 인증을 진행할 수 있다. 보통 UsernamePasswordAuthenticationFilter 클래스를 상속받은 `CustomAuthenticationFilter`를 생성한다.
   * &#x20;<mark style="color:orange;">`Token`</mark> 은 `Authentication` 인터페이스를 구현한 객체이다.
3. <mark style="color:orange;">`Token`</mark>을 <mark style="color:red;">`AuthenticationManager`</mark>로 전달한다.
4. <mark style="color:red;">`AuthenticationManager`</mark>는 전달받은 <mark style="color:orange;">`Token`</mark>을 <mark style="color:green;">`AuthenticationProvider`</mark>들로 전달해 실제 인증 과정을 시작한다. 이 과정에서 <mark style="color:orange;">`Token`</mark>으로부터 아이디를 조회해 `UserDetailsService`로 넘긴다.
5. `UserDetailsService` 에서는 아이디를 입력받아 DB에서 유저 데이터를 조회하고, 결과를 UserDetails 객체로 반환한다.
   * DB에서 유저 정보를 조회하는 로직은 서비스마다 다르기 때문에 Spring Security에서는 이 부분을 인터페이스로 구현해두었다. 따라서 `UserDetailsService` 를 구현하는 `CustomUserDetailsService` 클래스를 생성해 loadUserByUsername() 메서드로 유저 데이터를 조회하도록 구현한 후 <mark style="color:red;">`AuthenticationManager`</mark> 에 등록해야 한다.
6. \-
7. `UserDetailsService` 에서 아이디 기반으로 조회된 `UserDetails` 결과는 <mark style="color:green;">`AuthenticationProvider`</mark>로 반환된다.
8. <mark style="color:green;">`AuthenticationProvider`</mark> 에서는 `UserDetails` 객체와 토큰으로부터 입력받은 정보를 비교해 인증이 완료되면 <mark style="color:orange;">`인증된 Token`</mark> 객체를 생성해 <mark style="color:red;">`AuthenticationManager`</mark>로 반환한다.
9. <mark style="color:blue;">`AuthenticationFilter`</mark> 는 <mark style="color:orange;">`인증된 Token`</mark>을 받게 되면 `LoginSuccessHandler` 로 이동한다.
   * `CustomLoginSuccessHandler`
     * `CustomAuthenticationFilter` 에서 인증이 성공적으로 수행된 후에 처리될 Handler 클래스
       * <mark style="color:green;">`AuthenticationProvider`</mark>를 통해 인증이 성공될 경우 처리된다.
       * Bean으로 등록하고 SecurityConfig에서 `CustomAuthenticationFilter`의 핸들러로 추가해준다.
       * 로그인 성공 시 JWT 토큰 혹은 세션을 생성해 `HttpServletResponse` 에 담아 반환한다.
10. `LoginSuccessHandler` 에서는 <mark style="color:orange;">`인증된 Token`</mark> 객체를 스프링 시큐리티가 관리하는 SecurityContext에 저장한다.

## 클래스 별 설명

* 앞서 흐름별로 접했던 클래스들의 동작과 특징을 하나씩 알아본다.

#### **SecurityContextHolder**

* 보안 주체의 세부 정보를 포함하여 응용프로그램의 현재 보안 컨텍스트에 대한 세부 정보를 저장하는 클래스이다.
* 직접 Authentication 정보를 저장할 수 있다.

```java
SecurityContextHolder.getContext().setAuthentication(authentication);
```

* 직접 Authentication 정보를 조회할 수 있다. (하지만 @AuthenticationPrincipal 을 사용하는 것을 권장한다.)

```java
UserDetails authentication = (UserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
```

#### **SecurityContext**

* SecurityContextHolder가 관리하는 클래스로, `Authentication` 을 필드로 가지는 간단한 클래스이다.

#### Authentication

* **현재 접근하는 주체의 정보와 권한을 저장**하는 인터페이스이다.
* 이 객체는 `SecurityContextHolder`내에 저장된 `SecurityContext` 에 접근해 getAuthentication() 메서드로 조회할 수 있다.
* `Principal`을 상속한 객체이다.
  * principal은 일반적으로 UserDetails로 캐스팅된다.

#### UsernamePasswordAuthenticationToken

*   Authentication을 implements한 AbstractAuthenticationToken를 상속한 하위 클래스

    (단순히 말하면, **Authentication을 구현**했다고 보면 됨)
* User의 ID가 principal 역할을 하고, Password가 Credential 역할을 한다.
* 생성자에서 authorities 를 입력받으면, Authenticated된 객체를 생성한다.
* 생성자에서 authorities를 입력받지 못하면, Authenticated되기 전 객체가 생성된다.

#### **AuthenticationProvider**

* 실제 인증을 처리하는 인터페이스
* **authenticate()**: 인증 전의 Authentication 객체를 입력받아 인증된 Authentication 객체를 반환한다.
* 사용자는 Custom한 AuthenticationProvider을 작성해서 AuthenticationManager에 등록해야 한다.

#### **AuthenticationManager**

* AuthenticationManager에 등록된 AuthenticationProvider에 의해 인증 작업이 처리된다.
* 인증이 성공되면 **인증된** Authentication **객체를 Security Context에 저장**하고 인증 상태 유지를 위해 **세션에 저장**한다.
* 인증이 실패하면 AuthenticationException이 발생한다.

#### ProviderManager

* `List<AuthenticationProvider>`를 순회하면서 authenticate 작업을 처리한다.

#### **UserDetails**

* 인증에 성공하면 생성되는 객체
* UsernamePasswordAuthenticationToken을 생성하기 위해 사용
* 사용자는 User나 CustomUserDetails클래스를 작성해 UserDetails를 implements해야 한다.

#### UserDetailsService

* 사용자는 `UserDetailsService` 를 구현한 `CustomUserDetailsService` 클래스를 생성해야 한다.
* `loadUserByUsername` 메서드
  * `CustomUserDetailsService` 클래스는 UserDetails를 반환하는 이 메서드를 Override해야 한다.
  * 이 메서드 내부에서는 userRepository를 통해 가져온 User 객체를 UserPrincipal로 변환해 반환한다.
