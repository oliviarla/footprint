# 책임 연쇄 패턴

## 접근

* 1개 요청을 2개 이상의 객체에서 처리해야 할 때 컴포지트 형태 대신 체이닝 형태로 객체가 해당 요청을 검토하여 직접 처리하거나 다른 객체에 넘기도록 구현해야 한다.

## 개념

* **핸들러들의 체인​(사슬)​을 따라 요청을 전달**할 수 있게 하는 디자인 패턴이다. 영어로는 Chain of Responsibility Pattern 이라고 한다.
* 핸들러란 특정 행동들을 독립적으로 실행할 수 있는 객체를 의미한다.
* 각 핸들러는 요청을 받으면 **요청을 처리할지 아니면 체인의 다음 핸들러로 전달할지**를 결정한다.

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 모든 구상 핸들러에 공통적인 핸들러 인터페이스를 선언하고, 필요하다면 BaseHandler를 생성해 다음 핸들러의 참조를 저장하도록 한다. 핸들러 인터페이스의 구현체에는 요청을 확인한 후 처리할 지 다음 핸들러에 넘길 지 등 세부 로직이 담긴다.
* 클라이언트는 체인을 구성하여 필요한 요청을 보내면 된다.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt="" width="285"><figcaption></figcaption></figure>

* 컴포지트 패턴은 컴포넌트의 체인에 의해 트리의 루트 노드까지 도달할 수 있다는 점에서 책임 연쇄 패턴과 다르다.
* 데코레이터 패턴과 유사하게 실행을 일련의 객체들을 통해 재귀적인 합성에 의존하지만, 책임 연쇄 패턴의 핸들러들은 독립적으로 임의의 작업을 실행할 수 있고 데코레이터는 객체의 행동을 확장하며 흐름을 중단할 수 없다는 점에서 다르다.

## 장단점

* 장점
  * 요청을 보낸 쪽과 받는 쪽을 분리할 수 있고, 처리 순서를 제어할 수 있다.
  * 객체는 사슬 내부 구조나 각각의 처리 객체를 알 필요가 없다.
  * 사슬 내부 객체나 순서를 바꿔 역할을 동적으로 추가/제거하거나 변경할 수 있다.
* 단점
  * 사슬 끝까지 갔지만 아무 객체도 요청을 처리하지 않을 수 있어, 요청이 반드시 수행된다는 보장이 없다.
  * 실행 시에 과정을 살펴보거나 디버깅하기 어렵다.

## 사용 방법

* 핸들러 인터페이스를 선언하고, 요청을 처리하는 메서드 시그니처를 선언한다.
* 핸들러 구현 클래스를 만들고 처리 로직 혹은 다음 핸들러로 전달하는 로직을 작성한다.
* 클라이언트가 핸들러 체인을 조립하거나 미리 구축된 체인을 반환하는 팩토리 클래스를 만든다.
* 체인에 요청을 보낸다.

## 예시

### 스프링 필터

* 스프링 MVC에서 제공되는 필터는 체인 형태로 구성된다. 요청을 검증하여 부적절하다면 DispatcherServlet까지 도달하지 못하게 할 수 있다.

```java
public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {}
    
    void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
    
    default void destroy() {}
}
```

* 다음은 WebFilter 어노테이션을 통해 커스텀 필터를 등록하는 예시이다. Order 어노테이션을 통해 필터 적용 순서를 지정할 수 있다.

```java
@Order(1)
@WebFilter(urlPatterns = "/*")
public class MyCustomFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 요청이 Controller에 도달하기 전 수행할 작업
        System.out.println("Custom Filter: Request is being processed");

        // 필터 체인의 다음 단계로 요청을 전달
        chain.doFilter(request, response);

        // 응답이 클라이언트로 돌아가기 전에 수행할 작업
        System.out.println("Custom Filter: Response is being processed");
    }
}
```
