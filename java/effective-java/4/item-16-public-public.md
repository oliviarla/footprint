# item 16) public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## public 필드

* 인스턴스 필드들을 모아두기만 하는 클래스의 경우 데이터 필드에 직접 접근할 수 있어 캡슐화의 이점을 제공하지 못한다.
* API 수정 없이는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드 접근 시 부수 작업 수행이 불가하다는 단점이 존재한다.
* 단점을 해결하기 위해 private 필드로 바꾸고, **public 접근자를 추가**하도록 한다.

## public 접근자

```java
class Point {
  private double x;
  private double y;

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```

* 이렇게 변경하면 클라이언트는 반드시 접근자를 통해 필드 값을 얻을 수 있으므로 클래스 내부 표현 방식을 언제든 변경 가능하다.
*   package-private 클래스나 private 중첩 클래스이면 데이터 필드를 노출해도 무방하다. 클라이언트도 어차피 해당 클래스를 포함하는 이기 때문에, 패키지 바깥 코드는 손대지 않고 데이터 표현 방식을 바꿀 수 있다.

    패키지 안에서만 동작하는 코드
* public 클래스의 필드가 불변(public final)이라면, public 필드의 단점은 그대로이지만 불변식을 보장하기는 하지만 다음 릴리즈에서 내부 표현을 바꾸지 못하므로 권하지 않는다.
