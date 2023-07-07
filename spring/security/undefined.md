# 클래스 별 설명

## 🩵 Spring Security

#### **SecurityContextHolder**

* 보안 주체의 세부 정보를 포함하여 응용프로그램의 현재 보안 컨텍스트에 대한 세부 정보를 저장

#### **SecurityContext**

* `Authentication` 을 보관하고, 요청 시 반환해준다.

#### Authentication

* **현재 접근하는 주체의 정보와 권한을 저장**하는 인터페이스
* 이 객체에 접근하려면 `SecurityContextHolder`를 통해 `SecurityContext` 로 접근해 꺼내와야 한다.
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

## 🩵 JWT

#### TokenProvider

* 토큰을 제공해주는 클래스
* secretKey는 설정 파일에 등록해두고, 객체가 생성될 때 Base64로 인코딩해 사용한다.
* createToken(): 토큰을 생성
* getAuthentication(): token을 통해 UserId
* getUserId()
* resolveToken()
* validateToken()

#### TokenAuthenticationFilter

* 필터 인터페이스를 구현해야 한다.
*   필터 내부의 로직은 아래와 같다.

    1. HTTP 요청으로부터 토큰을 추출해낸다.
    2. 유효성 검사를 진행한다.
    3. 유효성 검사가 완료되면 `Authentication` 객체를 생성한다.
    4. 생성된 `Authentication` 객체를 SecurityContext에 저장한다.

    ```java
    SecurityContextHolder.getContext().authentication = jwtTokenProvider.getAuthentication(it)
    ```
