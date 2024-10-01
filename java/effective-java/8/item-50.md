# item 50) 적시에 방어적 복사본을 만들라

## 방어적 복사

* 자바는 메모리 충돌 오류에서 안전한 언어이지만 다른 클래스로부터의 침범을 막을 수 없다. 클라이언트가 불변식을 깨뜨리는 것을 막기 위해 방어적으로 프로그래밍해야 한다.
* 아래 예제를 보면 생성자에서 입력받은 Date 클래스를 그대로 사용하지 않고, **값이 똑같은 새로운 객체를 만들어** 내부에서 사용하거나 반환한다. 이를 방어적 복사(defensive copy)라고 한다.
* 유효성 검사는 방어적 복사본을 만든 후 시행되어야 한다. 그렇지 않다면 멀티스레딩 환경일 때 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다.

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + " after " + end);
    }

    public Date getStart() {
        return new Date(start.getTime());
    }

    public Date getEnd() {
        return new Date(end.getTime());
    }
}

```

* 생성자의 방어적 복사에 clone 메서드를 사용할 경우, 확장된 하위 클래스가 clone 메서드를 마음대로 구현할 수 있으므로 위험하다. 따라서 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.
* 접근자 메서드에서는 내부적으로 갖고 있는 클래스가 신뢰할 수 없는 하위 클래스가 아니라 Data 타입임이 확실하므로 방어적 복사에 clone을 사용해도 된다. 다만, **인스턴스를 복사하는 데는 생성자나 정적 팩토리를 쓰는 것이 좋다**.(item13)
* 위 예제의 경우, 자바 8이상으로 개발해도 된다면 Instant, LocalDateTime 또는 ZonedDateTime을 사용하는 것이 좋고, 이전 버전의 자바를 사용한다면 Date 참조대신 Date.getTime()의 반환값을 사용하면 좋다.
* 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관할 때 항시 그 객체가 잠재적으로 변경될 가능성을 고려해야 한다. 변경 될 수 있는 객체가 클래스에 넘겨진 후 문제없이 동작할 지를 따져보아야 한다.

## Lombok Getter는 방어적 복사 불가

* Lombok getter에서는 같은 객체를 계속해서 반환하도록 구현되어 있다.
* 아래와 같이 직접 수행해보면 같은 객체를 반환함을 알 수 있다.

```java
public class item50 {
  @Getter
  public static final class Temp {
    private final int id;

    public Temp(int id) {
      this.id = id;
    }
  }

  //getter를 직접 구현
  public static final class CustomGetter {
    private final Temp temp;

    public CustomGetter(Temp temp) {
      this.temp = new Temp(temp.getId());
    }
    public Temp getTemp() {
      return new Temp(temp.getId());
    }
  }

  //lombok의 getter를 사용
  @Getter
  public static final class LombokGetter {
    private final Temp temp;

    public LombokGetter(Temp temp) {
      this.temp = new Temp(temp.getId());
    }
  }

  public static void main(String[] args) {
    CustomGetter customGetter = new CustomGetter(new Temp(1));

    //custom getter를 사용 했을 경우에는 새로운 인스턴스 반환한다.
    System.out.println(customGetter.getTemp());
    System.out.println(customGetter.getTemp());
    System.out.println(customGetter.getTemp());

    LombokGetter lombokGetter = new LombokGetter(new Temp(1));

    //lombok getter를 사용 했을 경우에는 새로운 인스턴스 반환하지 않는다.
    System.out.println(lombokGetter.getTemp());
    System.out.println(lombokGetter.getTemp());
    System.out.println(lombokGetter.getTemp());

  }
}
```

<figure><img src="../../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

## 방어적 복사의 생략

* 방어적 복사에는 성능 저하가 따르고, 항상 쓸 수 있지도 않다. 호출자가 확실히 컴포넌트 내부를 수정하지 않는다면 방어적 복사를 생략할 수 있다. 그렇다 해도 호출자가 해당 매개변수 값을 수정하지 말아야 함을 명확히 문서화하는게 좋다.
* 메서드를 호출하는 클라이언트가 해당 매개변수 객체를 더 이상 직접 수정하지 않아야 함을 명시한다면 방어적 복사를 생략할 수 있다. 그리고 클라이언트가 건네주는 가변 객체의 통제권을 넘겨받는다고 기대하는 메서드나 생성자에도 그 사실을 확실히 문서화 해야 한다.
* 해당 클래스와 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때 방어적 복사 생략 가능하다.
* 래퍼 클래스의 특성상 클라이언트는 래퍼에 넘긴 객체에 직접 접근하여 불변식을 쉽게 파괴할 수 있지만, 그 영향을 오직 클라이언트 자신만 받게 되어 방어적 복사 생략이 가능하다.
