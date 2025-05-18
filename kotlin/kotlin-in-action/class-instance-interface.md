# 클래스, 객체, 인터페이스

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

## 생성자와 프로퍼티

### 클래스 초기화

* 코틀린은 주 생성자와 부 생성자를 분리한다.

#### 주 생성자

* 아래와 같이 간단하게 클래스를 선언할 때 클래스 이름 뒤에 오는 괄호로 둘러싸인 부분이 주 생성자이다.
* 주 생성자는 생성자 파라미터를 지정하고 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의한다.
* val은 해당 파라미터에 상응하는 프로퍼티가 클래스 내부에 생성된다는 의미를 가진다.

```kotlin
class User(val nickname: String)
```

* **초기화 블록**은 주 생성자와 함께 사용되며 클래스 객체가 만들어질 때 실행될 초기화 코드를 명시한다.
* 생성자 파라미터의 맨 앞에 밑줄을 달면 생성자 프로퍼티를 의미한다. 이를 통해 실제 프로퍼티와 생성자 프로퍼티를 구분 가능하다.
* 주 생성자의 파라미터는 프로퍼티를 초기화하는 식이나 초기화 블록에서만 사용 가능하다.

```kotlin
class User constructor(_nickname: String) {
    val nickname: String

    init {
        nickname = _nickname
    }
}
```

* 아래와 같이 생성자 파라미터에 기본값을 정의할 수 있다.

```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true)
```

* 객체를 생성하려면 아래와 같이 선언하면 된다. 자바와 달리 new가 필요하지 않다.

```kotlin
val user1 = User("cutie")
```

* 모든 생성자 파라미터에 기본값을 정의하면 컴파일러는 파라미터가 없는 생성자를 만들어준다. 그리고 기본값을 사용해 객체를 초기화한다. 이를 통해 파라미터가 없는 생성자가 필요한 라이브러리들과 쉽게 통합할 수 있다.
* 자식 클래스에서 부모 클래스의 생성자를 호출해야 할 때에는 아래와 같이 생성자 인자를 넘기면 된다.

```kotlin
open class User(nickname: String) { ... }

class TwitterUser(nickname: String) : User(nickname) { ... }
```

* 부모 클래스의 생성자에 아무 인자도 없다면 아래와 같이 빈 괄호를 항상 명시해주어야 한다.

```kotlin
class RadioButton : Button()
```

* 클래스 외부에서 객체 생성을 못하도록 막으려면 주 생성자에 private 변경자를 붙이면 된다.

```kotlin
class Secretive private constructor() {}
```

#### 부 생성자

* 코틀린은 디폴트 파라미터 값과 이름 붙은 인자 기능을 제공하므로  자바에 비해 생성자를 오버로딩하는 경우가 적다.
* 여러 가지 방법으로 객체를 초기화하도록 하려면 부 생성자의 제공이 필요한 경우가 있다.
* 아래와 같이 constructor 키워드를 이용해 부 생성자를 선언한다. 그리고 상위 클래스의 생성자 호출을 위해 super 키워드를 이용한다.
  * 자바에서는 자식 클래스에서 상위 클래스의 생성자를 호출할 때 생성자의 가장 윗줄에 super(...)를 선언해야 하는 것처럼 여기서는 콜론을 이용해 생성을 위임한다.
  * this()를 통해 클래스 내부의 다른 생성자에 생성을 위임할 수도 있다.

```kotlin
class MyButton : View {
    constructor(context: Context) : super(ctx) {
        // ...
    }

    constructor(context: Context, attr: AttributeSet) : super(ctx, attr) {
        // ...
    }

    constructor(context: Context, _name: String) : this(ctx) {
        name = _name
    }
}
```

### 프로퍼티 구현

* 인터페이스에 추상 프로퍼티를 선언할 수 있다. 이 때 프로퍼티를 뒷받침하는 필드나 게터 등의 정보가 들어가 있지 않으므로 상태를 저장하려면 하위 클래스에서 상태 저장을 위한 프로퍼티를 만들어야 한다.
* 아래와 같이 User에 선언된 추상 프로퍼티를 구현하는 다양한 방법이 존재한다.

```kotlin
interface User {
    val nickname: String
}

class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
    override val nickname: String // 필드에 값을 저장 안함
        get() = email.subStringBefore('@') // SubscribingUser().nickname 시 getter가 호출됨
}

class facebookUser(val accountId: Int) : User {
    override val nickname = getFacebookName(accountId) // 외부 함수를 이용해 필드를 초기화
}
```

* 인터페이스에 getter와 setter가 있는 프로퍼티도 선언할 수 있지만, getter/setter에서 상태를 저장하는 필드를 사용할 수는 없다. 아래와 같이 인터페이스를 선언하면 email 프로퍼티는 반드시 오버라이드해야 하고, nickName은 오버라이드하지 않아도 된다.

```kotlin
interface User {
    val email: String
    val nickName: String
        get() = email.substringBefore('@')
}
```

* setter/getter를 사용해 값을 변경하거나 읽을 때마다 특정 로직을 실행하도록 할 수 있다. 이를 위해 접근자 안에서 프로퍼티를 뒷받침하는 필드에 접근하는 `field` 식별자를 사용해야 한다.
* setter에서는 `field` 값을 수정할 수 있지만 getter에서는 조회만 가능하다.

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent())
            field = value
        }
}
```

```kotlin
val user = User("Alice")
user.address = "Elsenheimerstrasse 47, 80687 Muenchen" // 여기서 setter가 내부적으로 사용되므로 주소가 바뀌었다는 메시지가 출력됨
```

### 접근자(getter/setter)의 가시성 변경

* 접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같지만, 원하는 대로 가시성 변경자를 추가할 수 있다.
* 아래와 같이 setter를 비공개로 하여 내부적으로만 필드 값을 변경하도록 할 수 있다.

```kts
class LengthCounter {
    var counter: Int = 0
        private set

    fun addWord(word: String) {
        counter += word.length
    }
}
```

## 데이터 클래스

* 자바와 마찬가지로 코틀린의 클래스들은 toString, equals, hashcode 등의 메서드를 오버라이드해야 한다.
* 코틀린은 `==` 을 사용해 두 객체를 비교할 때 참조 동일성을 검사하지 않고 객체의 동등성을 검사하도록 내부적으로 equals 메서드를 호출한다.
* `is` 연산자를 사용해 자바의 instanceof와 같이 객체의 타입을 검사할 수 있다.
* equals()가 true를 반환하는 두 객체는 hashcode() 메서드의 결과도 동일해야 한다.
* 코틀린의 데이터 클래스를 사용하면 **equals, hashCode, toString 메서드를 자동으로 생성**해주므로 따로 구현할 필요가 없다.
* equals, hashCode 메서드는 **주 생성자에 나열된 모든 프로퍼티를 사용**하도록 한다.
* 내부적으로 var 프로퍼티를 선언할 수도 있지만 모든 프로퍼티를 읽기 전용으로 만들어 불변 클래스로 만드는 것을 권장한다.

```kotlin
data class Client(val name: String, val postalCode: Int)
```

* 해시 셋은 hashcode 메서드의 결과를 사용해 해싱을 하는데, 데이터 클래스를 담는 해시 셋에서 특정 데이터 클래스의 내부 필드가 변경되어 해시코드가 달라지면 문제가 발생할 수 있다.
* 코틀린은 copy 메서드를 제공하여 불변 데이터 클래스의 객체를 기반으로 일부 값만 변경한 새로운 객체를 만들 수 있도록 한다.

```kotlin
val lee = Client("kim", 1234)
println(lee.copy(postalCode=4321))
```

## 클래스 위임

* 하위 클래스가 상위 클래스의 메서드 중 일부를 오버라이드 할 경우, 상위 클래스의 세부 구현 사항에 의존하게 된다. 이 상황에서 만약 상위 클래스의 구현이 바뀌거나 새로운 메서드가 추가된다면 코드가 비정상적으로 동작할 가능성이 있다.
* 코틀린에서는 이러한 문제를 예방하기 위해 기본적으로 클래스를 final로 취급하고 open 변경자로 열어둔 클래스만 상속 가능하도록 한다.
* 데코레이터 패턴처럼 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야 할 때에는 기존 클래스와 동일한 새 인터페이스를 만들고 이를 구현한 데코레이터 클래스에 기존 클래스를 필드로 두고 사용해야 한다.
* by 키워드를 사용하면 인터페이스에 대한 구현을 다른 객체에 위임중이라는 사실을 명시할 수 있다.
* 동작을 변경하고 싶은 메서드의 경우만 오버라이드를 하면 되고, 나머지 오버라이드할 필요가 없는 메서드들은 기존 클래스로 위임된다. 즉, 컴파일러가 자동으로 코드를 만들어준다.

```kotlin
// BEFORE
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()
    
    override val size: Int get() = innerList.size
    override fun isEmpty(): Boolean = innerList.isEmpty
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun iterator(): Iterator<T> = innerList.iterator()
    // ...
}

// AFTER
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList<T>()
): Collection<T> by innerList {}
```

## object 키워드

* 클래스를 정의하는 동시에 객체를 생성한다.

### 객체 선언

* 싱글톤과 같이 인스턴스가 하나만 필요한 클래스가 필요할 때 객체 선언 기능을 이용하면 쉽게 구현할 수 있다.
* 아래와 같이 객체 선언을 하면 생성자 호출 없이 선언된 위치에서 바로 객체가 생성된다. 따라서 생성자 정의가 필요 없다.

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()
    
    fun calculateSalary() {
        fun (person in allEmployees) {
            // ...
        }
    }
}
```

* 객체 선언에 사용된 이름을 통해 내부 변수나 함수에 접근할 수 있다.

```kotlin
Payroll.allEmployees.add(Person(...))
Payroll.calculateSalary()
```

* 객체 선언 시 클래스나 인스턴스를 상속받을 수 있다.
* Comparator 구현은 두 객체를 인자로 받아 어느 객체가 더 큰지 알려주는 정수를 반환하며, 이는 클래스마다 하나씩만 있으면 되므로 아래와 같이 객체 선언을 사용하면 좋다.

```kotlin
object CaseInsensitiveFileComparator : Comparator<File> {
    override fun compare(file1: File, file2: File): Int {
        return file1.path.compareTo(file2.path, ignoreCase = true)
    }
}
```

```kotlin
CaseInsensitiveFileComparator.compare(File("/User"), File("/user"))
```

* 객체 생성을 제어할 수 없고 생성자 파라미터를 지정할 수 없기 때문에 단위 테스트를 하거나 소프트웨어 설정이 달라질 때 유연하게 사용할 수 없다.
* 클래스 안에서 객체를 선언할 수도 있다.

```kotlin
data class Person(val name: String) {
    object NameComparator : Comparator<Person> {
        override fun compare(p1: Person, p2: Person): Int =
            p1.name.compareTo(p2.name)
    }
}
```

* 자바에서 코틀린 객체 선언으로 만들어진 객체에 접근하려면 아래와 같이 INSTANCE 정적 필드를 사용하면 된다.

```java
CaseInsensitiveFileComparator.INSTANCE.compare(File("/User"), File("/user"));
```

### 동반 객체

* 코틀린은 자바의 static 키워드를 지원하지 않는다. 대신 패키지 수준의 최상위 함수와 객체 선언을 제공하여 정적 메서드, 정적 필드의 역할을 대신한다.
* 클래스의 인스턴스와 관계없이 호출할 수 있어야 하면서 클래스 내부 정보에 접근해야 하는 함수가 필요할 때에는 클래스에 중첩된 객체 선언의 멤버 함수로 정의해야 한다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 클래스에 중첩된 객체 선언의 멤버 함수란 아래와 같이 특정 내부 클래스에 만든 객체 선언에 속한 함수를 의미한다.

```kotlin
class OuterClass {
    object NestedObject {
        fun hello() {
            println("Hello from NestedObject!")
        }
    }
}
```

* 클래스 안에 정의된 객체에 companion이라는 식별자를 붙이면 동반 객체로 선언된다.
* 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있다. 따라서 private 생성자도 호출이 가능하다.
* 동반 객체가 선언된 클래스를 통해 동반 객체 내부 함수나 프로퍼티에 접근할 수 있다.

```kotlin
class A {
    val name = "A"
    companion object {
        fun bar() {
            println("Companion object called with: " + name)
        }
    }
}
```

```kotlin
A.bar()
```

* 동반 객체를 사용해 아래와 같이 팩토리 패턴을 구현할 수 있다. 주 생성자는 private으로 두고, 두 가지 팩토리 메서드를 제공한다. 팩토리 메서드 내부에서는 private 생성자를 호출해 인스턴스를 만든다.
  * 결국 java의 static 메서드를 대신하기 위해 아래와 같이 동반 객체에 메서드를 정의하는 것이다.

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) =
            User(email.substringBefore('@'))

        fun newFacebookUser(accountId: Int) =
            User(getFacebookName(accountId))
    }
}
```

```kotlin
val subscribingUser = User.newSubscribingUser("bob@gmail.com")
val facebookUser = User.newFacebookUser(4)
```

* 아래와 같이 이름을 붙이거나 인터페이스를 상속받거나 확장 함수와 프로퍼티를 정의하는 등 일반 객체처럼 사용 가능하다.
  *   이름 붙이기



      ```kotlin
      class Person(val name: String) {
          // ...
          companion object Loader {
              fun fromJson(jsonText: String): Person = ...
          }
      }
      ```



      동반 객체에 이름을 붙이면 `persion = Person.Loader.fromJson("...")` 처럼 접근할 수 있다.
  *   인터페이스 상속받기

      ```kotlin
      interface JsonFactory<T> {
          fun fromJson(jsonText: String): T
      }
      class Person(val name: String){
          companion object: JSONFactory<Person> {
              override fun fromJson(jsonText: String): Person = ...
          }
      }
      ```



      아래와 같이 동반 객체를 감싸는 클래스를 인터페이스의 인스턴스로 넘길 수 있다.\


      ```kotlin
      fun loadJson(factory: JSONFactory<T>): T {
          // ...
      }

      loadJson(Person)
      ```
* 자바에서는 동반 객체에 이름을 붙이지 않는다면 Companion이라는 정적 필드를 통해 동반 객체에 접근할 수 있다.

```java
person.Companion.fromJson("...");
```

* 클래스에 동반 객체가 선언되어있다면, 외부에서 확장 함수를 정의해 사용할 수 있다.

```kotlin
class Person(val firstName: String, val lastName: String) {
    companion object {
        // ...
    }
}

// 확장 함수 선언
fun Person.Companion.fromJSON(json:String): Person {
    ...
}
```

```kotlin
val person = Person.fromJSON("...")
```

### 익명 내부 클래스

* 코틀린에서는 익명 객체를 정의하여 자바의 익명 클래스처럼 사용할 수 있다.
* 아래와 같이 object 키워드를 통해 생성된 익명 객체를 사용할 수 있다. 함수를 호출하면서 인자로 넘기기 때문에 객체에 이름을 붙이지 않으므로 익명 객체라고 부른다.

```kotlin
window.addMouseListener(
    object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            // ...
        }
        override fun mouseEntered(e: MouseEvent) {
        }
    }
)
```

* 익명 객체를 변수에 저장하는 것도 가능하다.

```kotlin
val listener = object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }
    override fun mouseEntered(e: MouseEvent) {
    }
}
```

* 식이 포함된 함수의 final이 아닌 로컬 변수를 익명 객체 식 안에서 사용할 수 있다.

```kotlin
fun countClicks(window: Window) {
    var clickCount = 0
    window.addMouseListener(object: MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }
    })
}
```
