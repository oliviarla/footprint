# item 20) 추상 클래스보다는 인터페이스를 우선하라

## 인터페이스와 추상 클래스

* 자바 8 이후 인터페이스에 default method가 제공됨에 따라, 추상 클래스와 인터페이스 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.
* 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.
* 자바에서는 단일 상속만 지원 하기 때문에 한 추상 클래스를 상속받은 클래스는 다른 클래스를 상속받을 수 없다.
* 인터페이스는 구현해야할 메소드만 올바르게 구현한다면, 어떤 다른 클래스를 상속했던 간에 같은 타입으로 취급된다.

## 인터페이스의 장점

### **기존 클래스에도 손쉽게 새로운 인터페이스 구현 가능**

* 인터페이스가 요구하는 메서드를 추가하고 `implements` 구문만 추가하면 된다.
* 기존 클래스에 추상 클래스를 끼워넣기는 어렵다.
* 두 클래스가 같은 추상 클래스를 상속하길 원한다면 계층적으로 두 클래스는 공통인 조상을 가지게 된다. 만약 두 클래스가 어떠한 연관도 없는 클래스라면 클래스 계층에 혼란을 줄 수 있다.

### **mixin 정의에 적합**

* mixin: 대상 타입의 주 기능에 선택적으로 추가적인 기능을 혼합(mixed in)한다는 의미
* mixin 인터페이스: 어떤 클래스의 주 기능이외에 믹스인 인터페이스의 기능을 추가적으로 제공하게 해준다.
* Comparable은 자신을 구현한 클래스의 인스턴스끼리 순서를 정할 수 있다고 선언하는 믹스인 인터페이스이다.
* 추상 클래스로는 기존 클래스에 덧씌우는 것이 불가능해 mixin을 정의할 수 없다.

### **인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.**

* 계층구조가 없는 개념들은 인터페이스로 만들기 편하다.
* 예를 들어 아래와 같이 싱어송라이터는 Singer와 SongWriter 인터페이스를 모두 구현할 수 있고, 더 나아가 모두 확장하고 새로운 메서드를 추가한 인터페이스도 정의할 수 있다.

```java
public class People implements Singer, SongWriter {
    @Override
    public void Sing(String s) {

    }
    @Override
    public void Compose(int chartPosition) {

    }
}
```

* 추상 클래스로는 계층 구조를 표현하기 어려우며 조합 폭발이 일어날 수 있다.

> 조합 폭발: 계층을 엄격히 구분하기 어려운 클래스들의 계층구조를 만들기 위해 많은 조합이 필요해져 조합의 경우의 수가 폭발적으로 증가하는 현상

### **래퍼 클래스 관용구와 함께 사용 가능**

* ****기능을 향상시키는 안전하고 강력한 수단이 된다.
* 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 쉽게 깨진다.

### **디폴트 메서드**

* 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면 디폴트 메서드로 제공해 중복해서 구현하는 것을 방지할 수 있다.
* default 메서드 제공 시 상속하려는 사람을 위한 @implSpec을 작성하면 좋다.
* Object의 equals, hashcode 같은 메소드는 디폴트 메서드로 제공해서는 안 된다.
* 인스턴스 필드나 public이 아닌 정적 멤버를 가질 수 없다.
* 본인이 직접 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

### **추상 골격 구현 클래스**

* 디폴트 메서드의 경우 위와 같은 한계가 존재하기 때문에, 추상 골격 구현 클래스를 인터페이스와 함께 제공하면 인터페이스와 추상 클래스의 장점을 모두 얻을 수 있다.
* 인터페이스는 타입을 정의하고 필요한 일부 디폴트 메소드만 구현하고, 추상 골격 구현 클래스는 나머지 메소드들도 구현한다.
* 추상 골격 구현 클래스를 확장하면 인터페이스 구현이 대부분 완료된다.
* Collection 프레임워크의 AbstractList, AbstractSet과 같은 추상 클래스는 각각 List, Set 인터페이스의 추상 골격 구현 클래스이다.
* 추상 골격 구현 클래스에 **공통되는 메서드를 구현**해두면, 골격 구현을 **확장할 때 모든 메서드를 구현할 필요 없어** 메서드의 **중복 작성을 방지**할 수 있다.

```java
public abstract class AbstractHuman implements Human {
  @Override
  public void move() {
    System.out.println("걷다");
  }

  @Override
  public void seat() {
    System.out.println("앉다");
  }

  @Override
  public void process() {
    move();
    seat();
    attack();
  }
}

public class Thief extends AbstractHuman implements Human {
    @Override
    public void attack() {
        System.out.println("표창을 던진다");
    }
}

public class Wizard extends AbstractHuman implements Human {
    @Override
    public void attack() {
        System.out.println("마법봉을 휘두르다");
    }
}
```

* simulated 다중 상속: 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의해 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 방식
* 아래와 같이 CommonHuman을 상속받아야 하므로 AbstractHuman 골격 구현 클래스를 상속받지 못하는 경우, simulated 다중 상속 방식을 활용할 수 있다.

```tsx
public abstract class AbstractHuman implements Human {
  @Override
  public void move() {
    System.out.println("걷다");
  }

  @Override
  public void seat() {
    System.out.println("앉다");
  }

  @Override
  public void process() {
    move();
    seat();
    attack();
  }
}

public class CommonHuman {
    public void printHumanInformation() {
        System.out.println("human being, 2022");
    }
}

public class Thief extends CommonHuman implements Human {
  InnerHuman innerHuman = new InnerHuman();

  @Override
  public void move() {
    InnerHuman.move();
  }

  @Override
  public void seat() {
    InnerHuman.seat();
  }
  @Override
  public void attack() {
    InnerHuman.attack();
  }
  @Override
  public void process() {
    printHumanInformation();
    InnerHuman.process();
  }

  private class InnerHuman extends AbstractHuman {
  	@Override
  	public void attack() {
      	System.out.println("표창을 던진다");
  	}
  }
}
```
