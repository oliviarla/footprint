# 함수 정의와 호출

## 코틀린 만의 함수

### 컬렉션 생성 함수

* 코틀린은 자체 컬렉션을 제공하지 않는다. 자바의 컬렉션을 사용하기 때문에 자바에서 코틀린 함수를 호출하거나 코틀린에서 자바 메서드를 호출할 때 변환할 필요가 없다.
* 대신 코틀린에서는 아래와 같이 자바 컬렉션을 쉽게 생성하기 위한 함수를 제공한다.

```kotlin
val list = arrayListOf(1, 2, 3)
val set = hashSetOf(1, 2, 3)
val map = hashMapOf(1 to "one", 7 to "seven")
```

* 컬렉션에는 기본 toString 구현이 들어있지만 커스텀한 구분자와 prefix, postfix를 지정하고 싶은 경우 아래와 같은 함수를 만들 수 있다.

```kotlin
fun<T> joinToString(collection: Collection<T>,
                    seperator: String,
                    prefix: String,
                    postfix: String) : String {
    val result = StringBuilder(prefix)
    for((index, element) in collection.withIndex()) {
        if (index > 0) result.append(seperator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

### 이름 붙인 인자

* 코틀린에서는 **함수 호출 시 인자의 이름과 함께 값을 넣을 수 있다**. 단, **코틀린으로 작성된 함수에 한해서만 사용 가능**하다.

```kotlin
joinToString(collection, seperator = " ", prefix = " ", postfix = ".")
```

### 디폴트 파라미터 값

* 함수 정의 시 디폴트 파라미터 값을 지정하면 오버로드를 많이 하지 않아도 된다.
  * 자바에서 디폴트 파라미터를 가진 코틀린 함수를 호출할 때에는 모든 인자를 명시해야 한다. 만약 자바에서 사용하기 편리하도록 하려면 `@JvmOverloads` 어노테이션을 추가하여 오버로딩 함수들을 자동으로 생성되게 해야 한다.

```kotlin
fun<T> joinToString(collection: Collection<T>,
                    seperator: String = ", ",
                    prefix: String = "",
                    postfix: String = "") : String {
    // ...
}

@JvmOverloads
fun<T> joinToStringForJava(collection: Collection<T>,
                           seperator: String = ", ",
                           prefix: String = "",
                           postfix: String = "") : String {
    // ...
}
```

### 최상위 함수

* 코틀린에서는 함수 정의 시 반드시 클래스 안에 있지 않아도 된다. 즉, 자바처럼 정적 유틸 클래스를 두지 않아도 된다.
* 함수를 소스 파일의 최상위 수준에 위치시키면 패키지의 멤버 함수가 된다. 패키지 내에서는 그냥 사용하면 되고, 패키지 외부에서는 패키지를 임포트해 사용하면 된다.
* 아래와 같이 join.kt 파일을 작성하면 실제로 컴파일러는 새로운 클래스를 만들고 최상위 메서드들을 정적 메서드로 둔다.

```kotlin
package strings

fun joinToString() : String { ... }
```

```javascript
package strings;

public class JoinKt {
	public static String joinToString(...) { ... } 
}
```

* 만약 새로운 클래스 이름을 지정하고 싶다면 아래와 같이 어노테이션을 추가하면 된다.

```kotlin
@file:JvmName("StringFunctions")
package strings
fun joinToString(...): String {...}
```

### 최상위 프로퍼티

* 최상위 함수와 마찬가지로 최상위 프로퍼티도 정의 가능하다. 이 역시 새로운 클래스의 정적 필드로 저장된다.

```kotlin
var opCount = 0
fun perfumeOperation () {
    opCount++
}
```

* `val`의 경우 getter, `var`의 경우 getter/setter가 생성되며, 상수로 정의했는데 getter를 사용하는 것이 이상하면 `const val` 타입으로 정의하면 된다.

```kotlin
const val opCount = 0
```

```java
public static final String opCount = 0;
```

## 확장 함수와 확장 프로퍼티

### 확장 함수

* 어떤 클래스의 멤버 메서드처럼 호출할 수 있지만 클래스 외부에 선언된 함수
* 추가하려는 함수 이름 앞에 확장할 클래스 이름을 덧붙이면 된다. 클래스 이름은 **수신 객체 타입**이고 확장함수가 호출되는 대상 객체는 **수신 객체**이다.
* 아래 예제에서 수신 객체 타입은 String이고, 수신 객체는 함수를 호출한 인스턴스 객체(this)이다.

```kotlin
package strings

fun String.lastChar(): Char = this.get(this.length - 1)
```

* 함수 호출은 다음과 같이 한다. 결국 String 클래스에 새 메서드를 추가한 것 같은 느낌을 주면서 실제 String 클래스의 소스 코드에는 영향이 가지 않는다.

```kotlin
val c = "Kotlin".lastChar()
```

* 확장 함수 내부에서는 수신 객체의 메서드나 프로퍼티를 바로 사용할 수 있다. 단 private, protected에는 접근할 수 없다.

```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) { // this.withIndex()의 경우 수신 객체의 메서드이다.
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```

```kotlin
val list = arrayListOf("a", "b", "c")
println(list.joinToString(" "))
```

#### 확장 함수 import하기

* 확장 함수를 사용하려면 클래스나 함수와 마찬가지로 import 해주어야 한다. `*`를 사용할 수도 있고 `as` 키워드를 사용해 임포트한 것을 다른 이름으로 사용할 수 있다.

```kotlin
import strings.lastChar // or, import strings.*

val "Kotlin".lastChar()
```

```kotlin
import strings.lastChar as last

val "Kotlin".last()
```

#### 자바에서 호출

* 내부적으로 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메서드이다. 따라서 실행 시점 부가 비용이 들지 않는다.
* 확장 함수가 들어있는 파일 이름에 따라 확장 함수가 담긴 클래스 이름이 정해지므로, StringUtil.kt 파일에 정의하면 StringUtilKt 클래스를 통해 정적 메서드에 접근할 수 있다.

```java
char c = StringUtilKt.lastChar("Java");
```

#### 오버라이드 불가

* 확장 함수는 정적 메서드와 같은 특징을 가지므로 확장 함수의 하위 클래스에서 오버라이딩할 수 없다.
* 아래와 같이 View라는 부모 타입에 Button 이라는 자식 타입 객체를 대입한다면 View의 확장 함수가 호출된다.

```kotlin
open class View {
    open fun click() = println("View clicked")
}

class Button: View() {
    override fun click() = println("Button clicked")
}

fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a button!")

fun main(args: Array<String>) {
    val view: View = Button()
    view.showOff() // I'm a view!
}
```

* 확장 함수와 멤버 함수의 이름과 시그니처가 같으면 멤버 함수가 우선시되어 호출된다. 만약 우리가 개발한 라이브러리에 클라이언트가 자체적으로 확장 함수를 붙여 쓰고 있었고, 하필 라이브러리에 새로 추가한 함수가 해당 확장 함수와 이름, 시그니처가 같다면 클라이언트 코드에서 확장 함수 대신 새로 추가된 멤버 함수를 호출하도록 바뀔 것이다.

### 확장 프로퍼티

* 기존 클래스 객체에 대한 프로퍼티처럼 확장 프로퍼티를 등록할 수 있다.
* 일반적인 프로퍼티에서 수신 객체 클래스가 추가된 것이다. 기본 getter 구현을 할 수 없으므로 반드시 직접 정의해야 한다.
* 상태 값을 담을 수 없기 때문에 초기화 코드를 사용할 수 없다.

```kotlin
package strings

val String.lastChar: Char
    get() = get(this.length - 1)
```

* setter를 정의하면 변경 가능한 확장 프로퍼티를 만들 수 있다.

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
```

* 멤버 프로퍼티와 사용법은 똑같다.

```kotlin
println("Kotlin".lastChar)

val sb = StringBuilder("Kotiln?")
sb.lastChar = '!' // setter 사용
```

## 컬렉션 처리

#### 다양한 확장 함수의 제공

* 코틀린은 자바의 컬렉션을 사용하지만 더 확장된 API를 제공한다. 비결은 확장 함수를 사용하는 것이다.
* 예를 들어 자바 리스트에서 제공하지 않는 맨 마지막 원소 조회를 코틀린은 확장 함수를 통해 제공하고 있다.

```kotlin
fun <T> List<T>.last() : T { ... }
```

```kotlin
val list: List<String> = listOf("a", "b", "c")
print(list.last()) // c
```

### 가변 인자 함수

*   자바의 varargs 기능과 비슷하게 코틀린에도 `vararg`를 제공한다.

    * 자바

    ```java
    static <E> List<E> of(E... elements) {
        // ...
    }
    ```

    * 코틀린

    ```kotlin
    fun listOf<T> (vararg values: T) : List<T> { ... }
    ```
* 가변 인자로 배열을 넘겨줄 때에는 배열 앞에 스프레드 연산자 `*`를 명시해주면 배열의 각 원소가 가변 인자로 들어가게 된다.

```kotlin
val list = listOf("args1", *args)
println(list)
```

### 값의 쌍 다루기





###





