# 클래스, 객체, 인터페이스



코틀린의 선언은 기본적으로 public final이다.

코틀린의 중첩 클래스는 기본적으로 내부 클래스가 아니므로 외부 클래스에 대한 참조가 없다.

data 클래스를 제공하여 일부 표준 메서드를 자동으로 정의해준다.

언어 차원에서 제공하는 delegate 기능을 사용하면 위임을 위한 메서드를 직접 작성할 필요가 없다.

object 키워드를 사용해 클래스와 인스턴스를 동시에 선언할 수 있다.

## 클래스 계층 정의

### 인터페이스

* 상태(필드)가 들어갈 수 없다.
* 추상 메서드와 구현이 있는 메서드를 정의할 수 있다.
* 다음은 간단한 인터페이스 선언 방법이다. 디폴트 구현이 있는 메서드를 선언하려면 함수를 그냥 구현하면 된다.

```kotlin
interface Clickable {
    fun click()
    fun showoff() = println("I'm Clickable")
}
```

* 다음은 추상 메서드가 있는 인터페이스를 구현하는 방법이다. 클래스 이름 뒤에 `:` 을 붙이고 인터페이스 혹은 클래스 이름을 붙이면 구현 혹은 확장할 수 있다.
* override 변경자를 이용해 상위 인터페이스나 클래스의 메서드를 오버라이드할 수 있다. 메서드를 재정의 할 때에는 override를 반드시 명시해주어야 한다.

```kts
class Button : Clickable {
    override fun click() = println("I was clicked")
}
```

* 여러 인터페이스에 동일한 메서드 시그니처가 존재하고 이를 동시에 구현하는 클래스가 있다면, 어떤 인터페이스의 메서드를 사용할 지 오버라이드 메서드로 결정해야 한다.

```kotlin
class Button : Clickable, Forcusable {
    override fun click() = println("I was clicked")
    override fun showOff() {
    }
}
```





* 코틀린의 가시성, 접근 변경자를 아무것도 지정하지 않았을 때 자바와 다르다.
*
* sealed 변경자





