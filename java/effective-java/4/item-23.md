# item 23) 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 태그 달린 클래스

* 태그 달린 클래스는 장황하고, 오류가 쉽게 발생하고, 비효율적이다.
* 여러 구현이 한 클래스에 혼합되어 가독성이 안좋고 메모리도 많이 사용한다.
* 필드를 final으로 선언하려면 해당 의미에 쓰이지 않는 필드까지 생성자에서 초기화해야 한다.
* 새로운 의미를 추가할 때 마다 메서드를 변경(ex. area와 같은 메서드)하고 생성자를 추가해야 한다.
* 아래와 같이 하나의 클래스 내부에서 여러가지 도형을 나타내려 하면, 서로 필요없는 필드들이 혼재된다.

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
    	shape = Shape.CIRCLE;
	    this.radius = radius;
    }
    
    // 사각형용 생성자
    Figure(double length, double width) {
  	  shape = Shape.RECTANGLE;
 	   this.length = length;
 	   this.width = width;
    }
    
    // 도형의 면적 구하기
    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

## 클래스 계층구조화 하기

* 계층 구조의 루트가 될 추상 클래스를 정의하고 태그 값에 따라 동작이 달라지는 area같은 메서드들을 루트 클래스의 추상 메서드로 선언한다.
* 태그 값에 상관 없는 메서드들은 루트 클래스의 일반 메서드로 추가
* 모든 하위 클래스에서 공통으로 사용하는 필드를 루트 클래스의 필드로 추가
* 각 클래스의 생성자가 모든 필드를 초기화하고 추상 메서드를 구현했는지 컴파일러가 확인해주므로 런타임 오류 발생할 일이 없다.
* 타입 사이의 자연스러운 계층 관계를 반영해 컴파일타임 타입 검사 능력을 높여준다.

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```
