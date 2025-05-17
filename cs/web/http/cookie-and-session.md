# 쿠키와 세션

## 쿠키

### session cookie

* 사용자가 브라우저를 닫을 때 삭제되는 임시 쿠키
* 단일 브라우징 세션 동안 웹사이트에서 사용자 활동에 대한 정보를 유지하기 위해 일반적으로 사용
* 유저 정보를 안전하게 유지할 수 있다
* ex) 장바구니에 추가된 항목 또는 웹사이트 기본 설정 등

### persistent cookie

* 사용자가 브라우저를 닫은 후에도 사용자의 장치에 남아 있는 쿠키
* 만료 날짜가 있으며 여러 브라우징 세션에서 액세스할 수 있는 정보를 저장하는 데 사용
* ex) 로그인 자격 증명, 언어 기본 설정 및 기타 사용자 지정 설정 등

## 세션

### 개념

* stateless한 HTTP 통신에서 매번 로그인하지 않고도 클라이언트가 인증되었음을 확인하는 방법 중 하나이다.
* 서버는 클라이언트의 정보를 유지하기 위해 세션 메모리를 할당해 클라이언트의 인증정보를 관리함
* 클라이언트가 서버에 요청을 보내면, 서버는 세션 아이디를 쿠키에 담아 응답할 때 함께 전달
* 클라이언트는 세션 아이디를 담은 쿠키를 매 요청마다 함께 전달
* 매 요청마다 서버는 캐시, DB, 메모리 등에서 클라이언트로부터 받은 세션 아이디로 세션이 존재하는지 확인해야 하므로 token 방식에 비해 리소스가 든다.
* 이 때 사용되는 쿠키의 종류는 session cookie이며, 서버가 종료되거나 유효기간이 만료되거나 클라이언트 브라우저가 종료되면 제거된다.

<figure><img src="../../../.gitbook/assets/image (24) (1) (1) (1).png" alt=""><figcaption><p><a href="https://drsggg.tistory.com/388">https://drsggg.tistory.com/388</a></p></figcaption></figure>

### HttpSession

* 자바 서블릿에서는 HttpSession 객체를 통해 세션을 사용할 수 있다.
* 스프링에서 세션 방식을 제공하기 위한 표준화된 방식
* 세션 저장과 조회를 쉽게 할 수 있도록 해주며, 만료된 세션을 자동으로 관리해준다.
* SPRING\_SESSION, SPRING\_SESSION\_ATTRIBUTE로 구성
  * SPRING\_SESSION : 세션의 기본키와 ID, 생성 시간, 마지막 접속 시간, 유효 기간 등을 저장
  * SPRING\_SESSION\_ATTRIBUTE : 세션의 기본키와 어트리뷰트 이름, 바이트 형식의 직렬화된 객체를 저장
* getattribute(), setattribute()로 세션의 어트리뷰트에 접근할 수 있다

[https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpSession.html](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpSession.html)
