---
description: 회원 인증을 처리하는 방법 중 하나
---

# JWT 인증 방식

## 개념

* **JSON** 객체를 사용해서 토큰 자체에 정보들을 저장하고 있는 **Web Token**
* 유저를 인증하고 식별하기 위한 토큰 기반 인증
* 클라이언트에 토큰이 저장되므로 서버의 부담 감소
* 위변조 방지를 위해 개인키를 통한 전자서명 존재
* 클라이언트가 JWT를 서버로 전송 시 서버는 서명을 검증하고, 검증 완료 시 요청한 응답을 돌려줌

## 구조

* `Header`: JWT에서 사용할 토큰의 타입과 signature를 해싱하기 위한 해시 알고리즘 종류 저장
* `Payload`: 토큰에 담을 정보를 클레임 별로 저장
  * `Claims` : key-value 형태로 유저 정보에 대한 데이터를 저장하게 된다.
* `Signature`: 토큰의 유효성 검증을 위한 문자열이다. header의 인코딩 값, payload의 인코딩 값을 가운데에 `.`을 두고 합친 후, secret key로 해싱하고, base64로 인코딩해 저장

## 사용하는 이유

* 최근에는 MSA 환경이 많아져 각기 다른 서버에 요청을 보내는 경우가 많이 발생하는데, JWT의 경우 중앙의 인증서버, 데이터 스토어에 대한 의존성이 없어 편리하게 사용 가능하다.&#x20;
* session을 사용하는 경우에는 로그인 정보가 달라져 로그인이 풀릴 수 있다. 물론 redis 등의 공통 캐시 서버를 사용하면 되지만, 이 역시 비용이 발생할 수 있다. 성질이 다른 서버에서 공통으로 사용할 인증 로직으로 사용하기에는 적절하지 않을 수 있다.
* Base64 URL Safe Encoding을 사용하기 때문에 URL, Cookie, Header 에 모두 사용 가능하다.

## 단점

* Payload의 정보가 많아지면 네트워크 사용량 증가하기 때문에 데이터 설계 고려 필요
* 토큰이 각 클라이언트에 저장되므로 서버에서 각 클라이언트의 토큰 조작 불가

## 웹사이트 인증 방식

<figure><img src="../../.gitbook/assets/image (98).png" alt="" width="484"><figcaption><p>mechanism of JWT Authentication With HTTP</p></figcaption></figure>

* 로그인 창에서 username, password를 입력하면, 인증이 이뤄진 후 토큰을 생성해 클라이언트로 보낸다.
* 이제 이 토큰은 API에 대한 열쇠와 같은 역할을 한다. 토큰이 있다면 토큰과 함께 마이페이지의 내 정보 API 요청을 보내게 된다.
* 서버는 클라이언트로부터 받은 요청에 토큰이 있는지 확인하고, 토큰에 존재하는 권한을 체크한 후 API 요청 응답을 보낸다.

## 클래스 별 설명

#### TokenProvider

* 토큰을 제공해주는 클래스
* secretKey는 설정 파일에 등록해두고, 객체가 생성될 때 Base64로 인코딩해 사용한다.
* createToken(): 토큰을 생성
* getAuthentication()
* getUserId()
* resolveToken()
* validateToken()

#### TokenAuthenticationFilter

* 한 번 로그인한 사용자가 JWT로 인가를 받기 위해서는, 토큰을 확인하여 인증 처리하는 필터를 구현해야 한다.
* 필터 내부의 로직은 아래와 같다.
  1. HTTP 요청으로부터 토큰을 추출해낸다.
  2. 유효 시간, 위변조 확인 등 JWT 유효성 검사를 진행한다.
  3. JWT 유효성 검사가 완료되면 `Authentication` 객체를 생성한다.
  4. 생성된 `Authentication` 객체를 SecurityContext에 저장한다.
