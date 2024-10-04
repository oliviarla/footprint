# Spring MVC

## 스프링 MVC Lifecycle

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Filter

* 서블릿이 제공하는 기술
* Dispatcher Servlet이 실행되기 전 호출된다.
* 요청을 실질적으로 처리하기 전 처리해야 할 부분을 담당한다.
* 적절하지 않은 요청이 들어올 경우 HTTP 요청이 서블릿까지 도달하지 못하게 하는 역할을 한다.
* 체인으로 구성되며, 중간에 필터를 자유롭게 추가/제거 가능

#### 예시

* Authentication Filters
* Logging and Auditing Filters
* Image conversion Filters
* Data compression Filters
* Encryption Filters
* …

#### 인터페이스 구조

*   웹 관련 공통사항을 처리하기 때문에 파라미터로 ServletRequest와 ServletResponse 객체가 제공된다.

    > 스프링 프레임워크는 확장성을 위해 ServletRequest와 ServletResponse 객체로 제공했지만 대부분 HttpServletRequest와 HttpServletResponse를 사용하기 때문에 다운 캐스팅하여 사용하면 됨
* 메서드
  * **init() 메서드:** 필터 초기화 메서드이며 서블릿 컨테이너가 생성될 때 호출
  * **doFilter() 메서드:** 고객의 요청이 올 때마다 해당 메서드가 호출되며 메인 로직을 이 메서드에 구현하면 됨
  * **destory() 메서드:** 필터 종료 메서드이며 서블릿 컨테이너가 소멸될 때 호출

## Dispatcher Servlet

#### 개념

* Dispatcher → 배치 담당자 라는 뜻
* 들어오는 **모든 Request를 우선적으로 받아 처리해주는 서블릿**

#### 동작 순서

* HandlerMapping에게 Request에 대해 매핑할 Controller 검색을 요청 후 Controller 정보를 반환받는다.
* 실행할 Controller 정보를 HandlerAdapter에 넘기면, 컨트롤러가 실행된다.

## Handler Interceptor

#### 개념

* Spring MVC가 제공하는 기술
* 컨트롤러에 들어오는 요청 **HttpRequest**와 컨트롤러가 응답하는 **HttpResponse**를 가로챈다.
* Dispatcher Servlet이 실행된 후 호출되므로 **Request가 Controller에 매핑되기 전 앞단에서 부가적인 로직을 수행할 수 있다.**
* 웹 관련 공통 관심 사항을 처리한다는 점에서는 필터와 비슷
* 체인으로 구성되며 중간에 인터셉터를 자유롭게 추가/제거 가능

#### 예시

* 주로 세션, 쿠키, 권한 인증 로직에 많이 사용됨

#### 인터페이스 구조

```jsx
public interface HandlerInterceptor {
	boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	void postHandle(
			HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception;

	void afterCompletion(
			HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception;
}
```

* preHandle
  * 컨트롤러 호출 전 실행됨
  * 파라미터 `handler`는 컨트롤러 오브젝트이다.
  * 리턴값이 `true`이면 다음 인터셉터로 진행되고, `false`일 경우 다음 인터셉터들을 실행되지 못한다.
* postHandle
  * 컨트롤러 호출 후 실행됨
  * `ModelAndView`가 제공되므로 작업결과를 참조하거나 조작할 수 있다.
* afterCompletion
  * 모든 작업(뷰 렌더링)이 완료된 후 실행됨
  * try catch finally 절에서 finally처럼 무조건 호출됨

**출처**

[https://jaimemin.tistory.com/1887](https://jaimemin.tistory.com/1887)

[https://joont92.github.io/spring/HandlerMapping-HandlerAdapter-HandlerInterceptor/](https://joont92.github.io/spring/HandlerMapping-HandlerAdapter-HandlerInterceptor/)

[https://victorydntmd.tistory.com/176#recentComments](https://victorydntmd.tistory.com/176#recentComments)
