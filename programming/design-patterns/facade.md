# 퍼사드 패턴

## 접근

* 하나 이상의 클래스 인터페이스를 **단순하고 간단하게 하나의 인터페이스로 사용할 수 있도록** 해야 한다.
* 예를 들어 영화를 재생시키기 위해 조명 객체를 가져와 어둡게 하고, 스크린 객체를 가져와 스크린을 내리고, 프로젝터 객체를 가져와 프로젝터를 켜고 랩탑을 연결하고, 랩탑 객체를 가져와 영화를 플레이시키는 복잡한 과정이 필요하다. 이를 단순하게 하려면 퍼사드 패턴을 사용해야 한다. 퍼사드 클래스를 만들고 메서드에 위 과정들을 나열하면 된다.

```java
@AllArgsConstructor
public class HomeTheaterFacade {
    Light light;
    Screen secreen;
    Projector projector;
    Laptop laptop;
    
    public void watchMovie(String movieName) {
        light.dim(10);
        screen.down();
        projector.on();
        projector.wideScreenMode();
        laptop.play(movieName);
    }
    
    public void turnOffMovie() {
        light.on();
        screen.up();
        projector.of();
        laptop.stop();
        laptop.off();
    }
}
```

## 개념

* 서브시스템을 캡슐화하는 것이 아니라 서브시스템에 있는 인터페이스를 통합 인터페이스를 제공하는 패턴이다. 단순화된 인터페이스로 서브 시스템을 더 편리하게 사용하기 위해 사용되며, 클라이언트 구현과 서브시스템을 분리할 수 있다.
* 서브시스템에 있는 모든 구성요소 필드들에 접근 가능해야 한다.

> **최소 지식 원칙**
>
> * 객체 자체, 메소드에 매개변수로 전달된 객체, 메소드를 생성하거나 인스턴스를 만든 객체, 객체에 속하는 구성 요소에만 요청을 보내 달성할 수 있다.
> * 메서드 호출하여 얻은 객체에 메서드를 호출할 경우 알게되는 객체의 수가 많아지므로 최소 지식 원칙이 깨진다.
> * 원칙을 적용하다보면 메서드 호출을 처리하는 래퍼 클래스를 더 만들어야 하므로 시스템이 복잡해지고 성능이 떨어질 수 있다.

* 어댑터 패턴과 비교하자면, 어댑터 패턴은 새로운 인터페이스 구현체를 제공하여 클라이언트에서 필요로 하는 인터페이스를 적응시키는 용도로 사용되고, 퍼사드 패턴은 어떤 서브시스템에 대한 간단한 인터페이스를 제공하는 용도로 쓰인다. 어댑터는 일반적으로 하나의 객체만 래핑하는 반면 퍼사드는 많은 객체의 하위시스템과 함께 동작한다.
* 대부분의 경우 하나의 퍼사드 객체만 있어도 충분하므로 싱글톤으로 사용될 가능성이 높다.

## 장단점

* 장점
  * 복잡한 하위 시스템의 코드를 별도로 관리할 수 있다.
* 단점
  * 너무 많은 인터페이스, 클래스와 결합될 수 있다.

## 사용 방법

* 기존 클라이언트 코드에서 각 하위 시스템들이 이루고 있는 동작을 간단히 추상화할 수 있는지 확인한 후, 새로운 퍼사드 인터페이스를 생성해 복잡한 동작들을 담은 메서드를 구현한다.
* 클라이언트 코드에서는 하위 시스템들에 대한 의존을 없애고 새로운 퍼사드 인터페이스에 의존한다.

## 예시

* 퍼사드 패턴은 보통 서비스 레이어에서 자주 사용된다. 복잡한 로직을 단순화하기 위해 추상화하고 재사용가능하도록 만든다는 관점에서 DDD의 도메인 서비스 개념과 유사하다. 다만 퍼사드는 주로 애플리케이션의 여러 서비스를 감싸는 데 초점을 맞추고 있는 반면, 도메인 서비스는 도메인 내의 비즈니스 로직을 처리하는 데 중점을 둔다.

```java
@Service
public class MemberService {
    public Member findById(Long id) {
        // ...
    }
}

@Service
public class OrderService {
    public Order findById(Long id) {
        // ...
    }
}

@RequiredArgsConstructor
public class Facade {
    private final MemberService memberService;
    private final OrderService orderService;
    private final RecommendService recommendService;
    
    public TwoTypeProducts getOrderedProductsAndRecommendProducts(Long memberId) {
        Member member = memberService.findById(memberId);
        // ...
    }
}
```
