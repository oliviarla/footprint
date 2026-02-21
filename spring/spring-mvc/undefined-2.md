# 예외 처리

## **BasicErrorController**

* 개발한 Controller에서 발생한 예외를 따로 처리해주지 않는다면 아래 그림처럼 WAS로 에러가 흘러가게 되고, 이를 감지한 WAS는 Basic Error Controller로 요청을 보내 처리하게 된다.
* 에러가 처리되지 않고 WAS가 에러를 전달받게 되면, status는 항상 500이 된다.
* 별도의 에러 처리 전략을 통해 상황에 맞는 에러 응답을 제공해야 세밀한 제어 요구 사항을 반영할 수 있다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## HandlerExceptionResolver

* 예외 처리 전략을 추상화한 인터페이스
* Spring에서 고안한 “에러 처리”라는 공통 관심사(cross-cutting concerns)를 메인 로직으로부터 분리하는 예외 처리 방식이다.
* 발생한 Exception을 catch하고 HTTP 상태나 응답 메세지 등을 설정해 WAS에 에러가 전달되지 않도록 한다.

### 인터페이스 구조

```jsx
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, 
            HttpServletResponse response, Object handler, Exception ex);
}
```

* `Object handler`는 예외가 발생한 컨트롤러 객체이다.
* 4가지의 구현체가 Bean 으로 등록되어 있다.
  * `DefaultErrorAttributes`: 에러 속성을 저장하며 직접 예외를 처리하지는 않는다.
  * `ExceptionHandlerExceptionResolver`: 에러 응답을 위한 Controller나 ControllerAdvice에 있는 ExceptionHandler를 처리함
  * `ResponseStatusExceptionResolver`: Http 상태 코드를 지정하는 @ResponseStatus 또는 ResponseStatusException를 처리함
  * `DefaultHandlerExceptionResolver`: 스프링 내부의 기본 예외들을 처리한다.
  * ExceptionResolver들은 HandlerExceptionResolverComposite로 모아서 관리하게 된다.

### @ExceptionHandler

* 컨트롤러의 메소드에 @ExceptionHandler를 추가하거나, 클래스에 @ControllerAdvice나 @RestControllerAdvice를 어노테이션을 달면 적용된다.
* @ExceptionHandler에 의해 발생한 예외는 ExceptionHandlerExceptionResolver에 의해 처리된다.

### @ControllerAdvice / @RestControllerAdvice

* 여러 **컨트롤러**에 대해 전역적으로 ExceptionHandler를 적용하는 방식이다.
* 하나의 클래스로 모든 컨트롤러에 대해 전역적으로 예외 처리가 가능하다.
* 직접 정의한 에러 응답을 일관성있게 클라이언트에게 내려줄 수 있다.
* 별도의 try-catch문이 없어 코드의 가독성이 높아진다.

## Filter에서 발생한 예외 처리

* 에러 처리 전용 필터를 두어 모든 필터를 거치는 doFilter 메서드에서 발생한 예외를 처리할 수 있다.

```java
@Component
public class ExceptionHandlerFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) {
        try {
            filterChain.doFilter(request, response);
        } catch (CustomException e) {
            // 커스텀 에러 응답 생성
            setErrorResponse(response, e.getErrorCode());
        } catch (Exception e) {
            // 기타 예기치 못한 에러 처리
            setErrorResponse(response, ErrorCode.INTERNAL_SERVER_ERROR);
        }
    }

    private void setErrorResponse(HttpServletResponse response, ErrorCode errorCode) {
        response.setStatus(errorCode.getStatus().value());
        response.setContentType("application/json; charset=utf-8");
        // ObjectMapper 등을 사용하여 JSON 에러 바디 작성...
    }
}
```
