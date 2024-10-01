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
  * 이 때 구체적으로 상위 타입을 지정하여 메서드를 호출할 때 `super<상위 클래스 타입>.메서드()` 형태로 호출해야 한다.

```kotlin
class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")
    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

> 코틀린은 자바 6과 호환되도록 설계되었기 때문에 코틀린에서 디폴트 메서드를 가진 인터페이스를 자바로 변환할 때 자바 인터페이스의 디폴트 메서드를 사용하는 대신 일반 인터페이스와 정적 메서드가 들어있는 클래스를 조합해 구현된다. 따라서 자바에서는 코틀린 인터페이스의 디폴트 메서드에 의존할 수 없다.

### 상속 제어 변경자

* 자바에서는 상속 금지 클래스에 붙이는 `final` 키워드가 존재한다. 코틀린의 클래스와 메서드는 기본적으로 final이다. 따라서 어떤 클래스를 상속하거나 메서드, 프로퍼티를 오버라이드해야 한다면 `open` 변경자를 붙여야 한다.

| 변경자      | 오버라이드 여부 | 설명                             |
| -------- | -------- | ------------------------------ |
| final    | 불가       | 클래스 멤버의 기본 변경자                 |
| open     | 가능       | 오버라이드 대상 클래스/메서드에 필요한 변경자      |
| abstract | 필수       | 추상 클래스의 멤버에 붙일 수 있는 변경자        |
| override | 오버라이드 중  | 상위 클래스/메서드 등을 오버라이드할 때 붙이는 변경자 |

* 인터페이스 멤버의 경우 final, open, abstract 변경자를 사용할 수 없다. 이미 open 상태이며, 본문이 없으면 자동으로 abstract가 된다. final이 될 수는 없다.
* 부모 클래스나 인터페이스의 메서드, 프로퍼티를 오버라이드하는 경우 해당 메서드는 기본적으로 open 상태이다. 즉, override가 붙은 메서드나 프로퍼티는 open 상태이며 만약 해당 메서드나 프로퍼티를 하위 클래스에서 오버라이드하지 않도록 막으려면 final 키워드를 붙여주어야 한다.

```kotlin
open class RichButton : Clickable {
    final override fun click() { // ... }
}
```

* 스마트 캐스트의 경우 타입 검사 후 변경될 수 없는 변수를 다른 타입으로 변환하는데, 프로퍼티가 val이고 커스텀 접근자가 없어야 하며 final이어야 한다. 프로퍼티는 기본적으로 final이기 때문에 대부분의 프로퍼티를 스마트 캐스트에서 사용할 수 있다.

### 추상 클래스

* 코틀린에서도 abstract로 클래스를 선언해 추상 클래스를 만들 수 있다.
* abstract 추상 클래스는 인스턴스화할 수 없다.
* 추상 클래스 내부의 추상 멤버는 기본적으로 open 상태이며, 자식 클래스에서 override해야 한다.
* 추상클래스 내부의 추상이 아닌 함수는 기본적으로 final 상태이며, open 변경자를 명시한다면 오버라이드가 가능해진다.

```kotlin
abstract class Animated {
    abstract fun animate()
    
    open fun stopAnimating() { // open을 명시하여 오버라이드 가능
        // ...
    }
    
    fun animateTwice() { // 오버라이드 불가
    }
}
```

### 가시성 변경자

* 아무 변경자도 없는 경우 public 상태가 된다. 아무 변경자도 없을 때 package-private이 되는 자바와는 다르다.

| 변경자       | 클래스 멤버             | 최상위 선언             |
| --------- | ------------------ | ------------------ |
| public    | 모든 곳에서 볼 수 있다.     | 모든 곳에서 볼 수 있다.     |
| internal  | 같은 모듈 안에서만 볼 수 있다. | 같은 모듈 안에서만 볼 수 있다. |
| protected | 하위 클래스에서만 볼 수 있다.  | 적용 불가              |
| private   | 같은 클래스에서만 볼 수 있다.  | 같은 파일 안에서만 볼 수 있다. |

* 클래스의 확장 함수는 해당 클래스의 private, protected 멤버에 접근할 수 없다.
* 코틀린 코드를 자바 바이트코드로 컴파일할 때 private 클래스는 패키지 전용 클래스로 변경하고, internal 변경자는 public이지만 멤버의 이름을 이상하게 변경하고, protected 멤버를 java의 protected로 변경한다.

### 중첩 클래스

* 자바에서는 내부 클래스를 static으로 선언하지 않는다면 항상 바깥 클래스 인스턴스에 대한 참조를 묵시적으로 포함한다. 하지만 내부 클래스를 사용할 경우 GC가 안되는 문제 등이 발생할 수 있다. [item-24-static.md](../../java/effective-java/4/item-24-static.md "mention")&#x20;
* 코틀린에서는 기본적으로 내부 클래스여도 바깥 클래스 인스턴스에 대한 접근 권한이 없고, inner 변경자를 붙여야 접근 권한을 얻을 수 있다.
* 아래와 같이 작성하면 자바에서 정적 내부 클래스와 동일한 형태가 된다.

```kotlin
class Outer {
    class Inner {
        // ...
    }
}
```

* inner 변경자를 붙인 내부 클래스의 경우 `this@Outer` 형태로 바깥 클래스 인스턴스에 접근할 수 있다.

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

### sealed 클래스

* 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.
* sealed 클래스의 하위 클래스를 정의할 때는 반드시 상위 클래스 안에 중첩시켜야 한다.&#x20;
* sealed 클래스를 사용하면 아래와 같이 when 식에서 sealed 클래스의 모든 하위 클래스를 처리하기 때문에 디폴트 분기(else) 가 필요 없다. 하지만 sealed 클래스의 새로운 하위 클래스가 생겨나면 반드시 when 식을 변경해야 한다.

```kotlin
sealed class Expr {
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr): Int =
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)    
```

* sealed 클래스는 자동으로 open 상태이다.&#x20;







* sealed 변경자





