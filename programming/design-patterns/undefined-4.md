# 템플릿 메서드 패턴

## 접근

* 중복되는 코드가 많은 여러 클래스들 사이에서 중복을 없애려면, 중복되는 부분은 추상 클래스에 두고 변하는 부분은 추상 메서드로 두어 자식 클래스가 각자 작성하도록 하면 된다.
* 공통점을 최대한 뽑아내는 것이 포인트이다.&#x20;

## 개념

* 알고리즘(비즈니스 로직)의 각 단계를 정의하며 서브 클래스에서 일부 단계를 구현할 수 있도록 유도한다.
* **공통적으로 필요한 코드는 추상 클래스에 정의**되어 있으며 **개별 구현이 필요한 코드는 추상 메서드로 정의해두고 추상 클래스에서 사용**하기 때문에, **알고리즘의 템플릿을 만든다**고 보면 된다. 추상 메서드는 서브 클래스에서 반드시 정의해야 한다.
* 알고리즘이 추상 클래스에 모두 모여있으므로 한 부분만 고치면 된다.
* 새롭게 비슷한 클래스를 생성해야 할 때 쉽게 구현할 수 있다.

<figure><img src="../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

* **후크**란 추상 클래스에서 선언되지만 기본적인 내용만 구현되어 있거나 아무 코드도 구현되지 않은 메서드를 의미한다. 서브클래스는 이를 오버라이드해도 되고 하지 않아도 된다. 알고리즘의 특정 단계가 선택적으로 적용되어야 할 때 후크를 통해 적용 여부를 검사할 수 있으며, 이외에 다양한 상황에 사용할 수 있다.
* 고수준 구성 요소(템플릿 추상 클래스)가 저수준 구성 요소(서브 클래스)에 의존하도록 하고, 저수준 구성 요소는 고수준 구성 요소에 의존하지 않도록 디자인되어 순환 의존성을 방지한다.
* **템플릿 메서드 패턴**은 바꿔쓸 수 있는 행동을 캡슐화하고 어떤 행동을 사용할 지는 서브 클래스에 맡긴다. **전략 패턴**은 알고리즘의 어떤 단계를 구현하는 방법을 서브클래스에서 결정한다. **팩토리 메서드 패턴**은 클래스 객체 생성을 서브 클래스에 위임한다.

## 장단점

* 장점
  * 공통적인 알고리즘 단계에 변화가 있더라도 쉽게 변경 가능하다.
  * 서브 클래스에서 변경이 발생해도 다른 서브 클래스, 추상 클래스에 영향이 가지 않는다.
* 단점
  * 알고리즘의 단계가 너무 많아지면 서브 클래스들에서 구현하기 어려워진다.
  * 자식 클래스를 통해 디폴트 단계 구현을 억제하여 리스코프 치환 원칙을 위반할 수 있다.

## 사용 방법

* 알고리즘을 여러 단계로 나눌 수 있는지 확인한 후 중복되는 단계는 추상 클래스의 일반 메서드로 구현해두고, 상황에 따라 달라질 수 있는 단계는 추상 클래스의 추상 메서드로 둔다. 그리고 추상 클래스에는 알고리즘의 순서에 맞게 메서드들을 호출하는 메서드를 둔다.
* 추상 클래스에 대한 서브 클래스들을 상황에 맞게 구현한다.

## 예시

* 홍차와 커피 우리는 방법에서 동일한 부분은 추상 클래스에 두고, 다른 부분은 각각 서브 클래스에서 구현하도록 한다.

```java
public abstract class CaffeinBeverage {
    final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }
    
    abstract void brew();
    
    abstract void addCondiments();
    
    void boilWater() {
        System.out.println("물 끓이기");
    }
    
    void pourInCup() {
        System.out.println("컵에 따르기");
    }
}

public class BlackTea extends CaffeinBeverage {
    public void brew() {
        System.out.println("찻잎을 우리기"); 
    }
    public void addCondiments() {
        System.out.println("레몬 얹기");
    }
}

public class BlackTea extends CaffeinBeverage {
    public void brew() {
        System.out.println("필터로 커피 우리기"); 
    }
    public void addCondiments() {
        System.out.println("설탕과 우유 추가하기");
    }
}
```

* Spring의 HTTP 통신을 위한 AbstractClientHttpRequest 클래스에서는 header, cookie 적용을 서브 클래스에서 구현하도록 한다. applyAttributes는 후크에 해당하는 메서드로, HTTP 요청에 attributes를 추가해야 하는 서브 클래스일 때에만 구현하면 된다.

```java
  public abstract class AbstractClientHttpRequest implements ClientHttpRequest {
    protected Mono<Void> doCommit(@Nullable Supplier<? extends Publisher<Void>> writeAction) {
      // ...

      this.commitActions.add(() ->
          Mono.fromRunnable(() -> {
            applyHeaders();
            applyCookies();
            applyAttributes();
            this.state.set(State.COMMITTED);
          }));

      // ...
      

      return Flux.concat(actions).then();
    }

    protected abstract void applyHeaders();

    protected abstract void applyCookies();
    
    protected void applyAttributes() {
    }

  }
```

```java
class JdkClientHttpRequest extends AbstractClientHttpRequest {
    // ...
    @Override
    protected void applyHeaders() {
      // ...
    }

    @Override
    protected void applyCookies() {
      MultiValueMap<String, HttpCookie> cookies = getCookies();
      if (cookies.isEmpty()) {
        return;
      }
      this.builder.header(HttpHeaders.COOKIE, cookies.values().stream()
          .flatMap(List::stream).map(HttpCookie::toString).collect(Collectors.joining(";")));
    }
}
```

```java
class ReactorNetty2ClientHttpRequest extends AbstractClientHttpRequest implements ZeroCopyHttpOutputMessage {
  @Override
  protected void applyHeaders() {
    getHeaders().forEach((key, value) -> this.request.requestHeaders().set(key, value));
  }

  @Override
  protected void applyCookies() {
    getCookies().values().forEach(values -> values.forEach(value -> {
      DefaultHttpCookiePair cookie = new DefaultHttpCookiePair(value.getName(), value.getValue());
      this.request.addCookie(cookie);
    }));
  }

  @Override
  protected void applyAttributes() {
    if (!getAttributes().isEmpty()) {
      ((ChannelOperations<?, ?>) this.request).channel()
          .attr(ReactorNetty2ClientHttpConnector.ATTRIBUTES_KEY).set(getAttributes());
    }
  }
}
```
