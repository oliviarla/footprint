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

* 중위 호출(infix call) 방식은 인자가 하나뿐인 메서드나 확장 함수에서 사용할 수 있으며, 일반 메서드 이름을 수신 객체와 유일한 메서드 인자 사이에 넣어 사용하는 방식이다.
* 아래 두 호출은 동일한 결과를 만든다. 아래와 같이 1, "one"을 to 메서드를 통해 Pair에 담은 후 다시 각각의 변수에 담는 것을 **구조 분해**라고 한다.

```kotlin
val (number, name) = 1.to("one")
val (number, name) = 1 to "one"
```

* 아래와 같이 withIndex를 구조 분해 선언과 조합하여 컬렉션 원소의 인덱스와 값을 따로 변수에 담을 수 있다.

```kotlin
for ((index, element) in collection.withIndex()) {
  println("$index: $element")
}
```

* 함수를 중위 호출에 사용될 수 있도록 하려면 선언 시 infix 변경자를 함수 선언 앞에 추가해야 한다.
* 아래는 중위 호출이 가능한 확장 함수인 to 함수로, 타입과 상관 없이 임의의 순서쌍을 만들 수 있다.

```kotlin
infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

## 문자열과 정규식

* 코틀린의 문자열은 자바의 문자열과 같으므로 변환이 필요 없다.
* 코틀린은 문자열에 대한 다양한 확장 함수를 제공한다. 예를 들어 문자열을 분리하는 split 확장 함수들을 제공하여 다양한 조합의 파라미터를 받을 수도 있고, substringBeforeLast, substringAfterLast 확장 함수를 제공하여 가장 마지막에 나타난 문자의 앞 / 뒤 문자열을 얻을 수 있다.
* 정규식을 파라미터로 받는 함수는 Regex 타입을 입력받는다.&#x20;
* 아래는 `.` 또는 `-` 을 기준으로 문자열을 분리하도록 한 예제이다.

```kotlin
"12.345-6.A".split(".", "-") // [12, 345, 6, A]

"12.345-6.A".split("\\.|-".toRegex()) // [12, 345, 6, A]
```

### 3중 따옴표 문자열

* 코틀린에서 3중 따옴표 문자열을 사용하면 정규식에서 어떤 문자도 이스케이프할 필요가 없으며 줄바꿈이 들어있는 텍스트를 쉽게 문자열로 만들 수 있다.
* 단, \n과 같은 특수 문자를 사용할 수 없다.
* 아래는 정규식에서 `\\.` 대신 `\.`을 사용하는 예시이다.

```kotlin
val regex = """(.+)/(.+)\.(.+)""".toRegex()
val matchResult = regex.matchEntire(path)
```

* 여러 줄 문자열을 표현할 때 들여쓰기와 줄바꿈을 사용했지만 `.`을 넣어두면, 실제 사용할 때 trimMargin 함수를 사용해 들여쓰기와 `.` 을 제거할 수 있다.

```kotlin
val kotlinLogo = """| //
                   .|//
                   .|/ \"""
println(kotlinLogo.trimMargin("."))
// | //
// |//
// |/ \
```

## 로컬 함수와 확장

* 함수에서 추출한 함수를 원래 함수 내부에 중첩시킬 수 있다. 이를 통해 작게 나누어진 메서드 간의 관계 파악의 어려움이 없어 코드 이해가 편리해질 수 있다.
* 로컬 함수는 자신이 속한 바깥 함수의 파라미터와 지역 변수를 사용할 수 있다.
* 일반적으론 한 단계의 함수만 중첩시켜 깊이가 깊어지지 않도록 하는 것이 권고된다.
* 아래는 사용자를 DB에 저장하기 전에 검증하는 과정을 로컬 함수로 분리한 것이다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {

    fun validate(value: String,
                 fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("Can't save user ${user.id}: empty $fieldName")
        }
    }

    validate(user, user.name, "Name")
    validate(user, user.address, "Address")

    // Save user to the database
}
```
