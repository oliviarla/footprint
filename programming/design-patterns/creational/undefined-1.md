# 프로토타입 패턴

## 접근

* 클래스의 객체를 생성할 때 자원과 시간이 많이 들거나 복잡한 경우 기존 객체를 복사하여 새로운 객체를 만들 수 있도록 한다.
* 클라이언트 코드에서는 어떤 클래스의 객체를 만드는지 몰라도 새로운 객체를 만들 수 있다.

## 개념

* 복제 대상인 객체에 복제 로직을 위임한다.
* 복제 기능을 지원하는 클래스들에 대한 공통 인터페이스를 선언하여 새로운 객체를 만들고 필드 값만 동일하게 설정하는 clone 메서드를 제공하도록 하는 디자인 패턴이다.
* 프로토타입이란 복제를 지원하는 객체를 의미한다. 미리 여러 프로토타입 객체들을 만들어두고 비슷한 객체가 필요한 경우 새로운 객체를 생성하는 대신 프로토타입을 복제하도록 한다.
* 아래는 프로토타입 패턴의 구성 요소이다.
  * **프로토타입 인터페이스**는 복제 메서드를 선언한다. 보통은 clone 메서드만을 가진다.
  * **프로토타입 구체 클래스**는 복제 메서드를 구현하는 클래스이다.
  * **클라이언트**는 프로토타입 인터페이스를 따르는 객체의 복사본을 손쉽게 생성할 수 있다.

<figure><img src="../../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

* **프로토타입 레지스트리 클래스**를 사용해 프로토타입 인터페이스 구현체들, 즉 미리 만들어진 객체들을 저장해둘 수 있다.

<figure><img src="../../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

## 장단점

* 장점
  * 클라이언트는 새로운 객체를 만드는 과정을 몰라도 되고 구체적인 클래스타입을 모르더라도 객체 생성이 가능하므로 단일 책임 원칙을 지킨다.
  * 객체를 새로 생성하는 것보다 복사하는 것이 효율적일 때 유용하다.
* 단점
  * 순환 참조가 있는 객체 등 복제가 까다로운 객체에는 적용하기 어려울 수 있다.

## 예시

* 아래와 같이 도형을 나타내는 Shape 추상 클래스가 있다. 이를 구현하는 클래스들은 clone 추상 메서드를 구현해야 한다.

```java
public abstract class Shape {
    public int x;
    public int y;
    public String color;

    public Shape() {
    }

    public Shape(Shape target) {
        if (target != null) {
            this.x = target.x;
            this.y = target.y;
            this.color = target.color;
        }
    }

    public abstract Shape clone();

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Shape)) return false;
        Shape shape2 = (Shape) object2;
        return shape2.x == x && shape2.y == y && Objects.equals(shape2.color, color);
    }
}
```

* 복사 생성자를 사용해 clone 기능을 구현한다. 자바에서는 Cloneable 인터페이스를 제공하기도 하지만, 복사 생성자나 복사 팩토리 형태를 권장한다. [item-13-clone.md](../../../java/effective-java/3/item-13-clone.md "mention")

```java
public class Circle extends Shape {
    public int radius;

    public Circle() {
    }

    public Circle(Circle target) {
        super(target);
        if (target != null) {
            this.radius = target.radius;
        }
    }

    @Override
    public Shape clone() {
        return new Circle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Circle) || !super.equals(object2)) return false;
        Circle shape2 = (Circle) object2;
        return shape2.radius == radius;
    }
}
```

```java
public class Rectangle extends Shape {
    public int width;
    public int height;

    public Rectangle() {
    }

    public Rectangle(Rectangle target) {
        super(target);
        if (target != null) {
            this.width = target.width;
            this.height = target.height;
        }
    }

    @Override
    public Shape clone() {
        return new Rectangle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Rectangle) || !super.equals(object2)) return false;
        Rectangle shape2 = (Rectangle) object2;
        return shape2.width == width && shape2.height == height;
    }
}
```
