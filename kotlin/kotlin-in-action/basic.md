# 코틀린 기초

## 함수

* 함수를 최상위 수준에 정의할 수 있다. 즉, 자바와 달리 반드시 클래스 안에 함수를 넣지 않아도 된다.
* `fun 함수이름(파라미터 목록): 반환타입` 형태로 함수를 선언할 수 있다.

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

> statement(문) vs expression(식)
>
> * 문은 자신을 둘러싸는 가장 안쪽 블록의 최상위 요소로 존재하며 아무런 값을 만들어내지 않는다.
> * 식은 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여할 수 있다.
> * 자바에서는 모든 제어 구조가 statement이지만, 코틀린에서는 루프를 제외하고는 모두 expression이다. 예를 들어 코틀린의 if는 식이기 때문에 결과를 만들어낸다.

* **블록이 본문인 함수**는 본문이 중괄호로 둘러싸여 있으며, **식이 본문인 함수**는 등호와 식으로만 이루어져 있다. 코틀린에서는 식이 본문인 함수가 자주 쓰인다.
* 식이 본문인 함수는 컴파일러가 함수 본문 식을 분석해 결과 타입을 함수의 반환 타입으로 정해주기 때문에, 함수의 반환 타입을 지정하지 않아도 된다.

## 변수

* 자바는 변수 선언 시 타입이 가장 먼저오지만, 코틀린은 타입을 생략하는 경우가 많다.
* 코틀린은 변수 이름 뒤에 타입을 명시하거나 생략할 수 있다.

```kotlin
val question = "밥을 먹었나요?"
val answer = "네"
val answer: Int = 1
```

* 변수의 선언과 동시에 초기화를 하지 않는다면 반드시 변수 타입을 명시해야 한다.
* 변경 불가능한(immutable) 변수는 `val` 로 선언하고, 변경 가능한 변수(mutable)는 `var` 로 선언한다. 기본적으로 `val` 로 선언하고 꼭 필요할 때에만 `var` 로 선언하는 것을 권장한다.
* `var` 로 선언하면 변수의 값은 변경 가능하지만, 변수의 타입은 변경이 불가능하다.
* `val` 로 선언하더라도 객체의 내부 값은 변경될 수 있다.

```kotlin
val language = arrayListOf("Java")
language.add("Kotlin")
```

* 문자열 템플릿
  * 문자열 리터럴 내부에서 변수를 사용할 수 있다.
  * `"Hello, $name!"` 과 같이  name이라는 변수를 문자열 내부로 가져올 수 있다.
  * `$` 문자를 문자열 내부에서 사용하려면 `"price: \$ 10"` 와 같이 사용해야 한다.
  * 변수는 가급적 중괄호로 감싸 사용하는 것이 한글 사용이나 정규식, 일괄 변환, 복잡한 식 사용 시에도 유리하다.
    * `Hello, ${if (args.size > 0) args[0] else "someone"}!"`

## 클래스

* 코틀린의 기본 가시성은 public이므로 변경자를 생략해도 된다.
* 아래는 코드 없이 데이터만 저장하는 클래스인 값 객체이다.

```kotlin
class Person(val name: String)
```

* 자바에서는 필드와 접근자를 묶어 프로퍼티라고 부르며, 코틀린은 이를 언어 기본 기능으로 제공하여 자바의 필드와 접근자 메서드를 대신한다.
* 클래스에서 프로퍼티 선언하는 방법은 변수를 선언하는 방법과 같다.
* `val`로 선언하면 private 필드와 public getter를 만들어내고, `var`로 선언하면 private 필드와 public getter/setter를 만들어낸다.
* 코틀린에서는 getter/setter 접근을 위해 getXXX, setXXX 메서드를 사용하는 것이 아니라 직접 프로퍼티에 접근하는 것처럼 사용하면 된다.

```kotlin
val person = Person("Bob", true)
println(person.name)
println(person.isMarried)
```

* 커스텀 접근자를 작성하여 프로퍼티에 접근하도록 만들 수 있다.

```kotlin
class Rectangle(val height: int, val width: int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}

// ...

println(rectangle.isSquare)
```

## 패키지 / 디렉토리

* 모든 코틀린 파일은 맨 앞에 package문을 넣을 수 있다.
* package문을 넣으면 파일 안에 있는 모든 선언이 패키지에 들어가, 같은 패키지에 속한 다른 파일에서 정의한 선언도 사용할 수 있다.
* 다른 패키지에 있는 선언을 사용하려면 import문으로 불러와야 한다.
* 함수를 import해 사용할 수도 있다.
* `import <패키지명>.*` 형태로 사용하면 최상위에 정의된 함수나 프로퍼티까지 모두 불러오게 된다.
* 코틀린에서는 여러 클래스를 한 파일에 넣을 수 있고 디렉토리에 관계없이 패키지를 구성할 수 있다.
* 하지만 자바에서는 반드시 패키지 구조와 일치하는 디렉토리에 클래스를 넣어주어야 한다.

## enum과 when

* enum은 **소프트 키워드**로, enum class로 정의해야 하며 **enum 이라는 문자 자체는 변수명으로 사용 가능**하다.
* 프로퍼티와 메서드를 추가하는 경우 enum 상수 목록과 메서드 정의 사이에 반드시 세미콜론이 필요하다.

```kotlin
// 가장 단순한 enum 선언
enum class Color {
    RED, ORANGE,
    YELLOW, GREEN, BLUE,
    INDIGO, VIOLET
}

// 프로퍼티와 메서드가 있는 enum 선언
enum class Color(
        val r: Int, val g: Int, val b: Int
) {
    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

* when을 사용해 enum타입에 따라 값을 매핑해 반환할 수 있다.

```java
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
        Color.BLACK, Color.WHITE -> "Colorless"
    }

val string = getMnemonic(Color.Blue) // "Battle"
```

* enum 상수 값을 임포트하여 짧은 이름으로 사용 가능하다.

```kotlin
import ch02.colors.Color
import ch02.colors.Color.*

fun getWarmth(color: Color) = 
    when (color) {
        RED, ORANGE, YELLOW -> "Warm"
        GREEN -> "neutral"
        else -> "cold"
    }
```

* when 식의 인자로 아무 객체나 사용할 수 있다. 아래는 코틀린에서 기본적으로 제공하는 setOf 함수를 사용해 두 Color 상수를 Set으로 만들어 매칭되는 Color를 반환하는 함수 예제이다.

```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("dirty color")
    }
mix(BLUE, YELLOW)
```

* when 식의 인자를 받지 않을 수도 있다. 이렇게 할 경우 앞서 setOf를 사용해 객체를 만들어 비교하는 것보다 성능, 메모리 관점에서 이득일 수 있다.

```kotlin
fun mix(c1: Color, c2: Color) =
    when {
        (c1 == RED && c2 == YELLOW) -> ORANGE
        (c1 == BLUE && c2 == YELLOW) -> GREEN
        else -> throw Exception("dirty color")
    }
```

### 스마트캐스트

* 타입 검사와 타입 캐스트를 조합한 것으로 `is` 를 사용해 타입을 검사하며, 이 때 컴파일러가 타입 캐스팅을 자동으로 수행해준다.

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```

```kotlin
fun eval(e: Expr): Int {
    if (e is Num) {
        // 코틀린에서는 as를 이용하여 타입 변환을 할 수 있지만 여기서는 불필요하다.
        // val n = e as Num
        // return n.value
        return e.value
    }
    if (e is Sum) {
        return eval(e.right) + eval(e.left)
    }
}
```

* 클래스의 프로퍼티에 대해 스마트 캐스트를 사용한다면 프로퍼티는 반드시 `val` 이어야 하며  커스텀 접근자를 사용하면 안된다. 왜냐하면 해당 프로퍼티에 대한 접근이 항상 같은 값을 반환하지 않을 수 있기 때문이다.

### if와 when

* 코틀린의 if는 값을 만들어내기 때문에 아래와 같이 if 문을 본문으로 한 함수를 쉽게 사용할 수 있다. if문에 블록을 사용하는 경우, 블록의 마지막 식이 결과 값으로 반환된다.

```kotlin
fun eval(e: Expr) Int =
    if (e is Num) {
        e.value
    } else if (e is Sum) {
        eval(e.right) + eval(e.left)
    } else {
        throw IllegalArgumentException("Unknown Expression")
    }
```

* if문의 블록에 식이 하나밖에 없다면 중괄호를 생략해도 된다. when을 사용하면 더욱 간략하게 표현할 수 있다.

```kotlin
fun eval(e: Expr): Int =
    when(e) {
        is Num -> e.value
        is Sum -> evalu(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

* 식이 본문인 함수는 블록을 본문으로 가질 수 없다. 블록이 본문이고 반환 타입이 있는 함수는 내부에 반드시 return문이 있어야 한다.

## 이터레이션

### while 루프

* while, do-while을 제공하며 문법은 자바와 다르지 않다.

```kotlin
while (조건) {
    // ...
}

do {
    // ...
} while(조건)
```

### 범위 표현

* 코틀린에서는 범위를 나타내기 위해 `..` 연산자를 사용하며 닫힌 구간을 나타낸다. 즉, 마지막값도 범위에 포함된다.

```kotlin
for (i in 1..100) {
    print(i)
}
```

* 다음과 같이 `in` 연산자와 `downTo`, `step`을 사용하면 100부터 1까지 2씩 이동하며 이터레이션 할 수 있다.

```kotlin
for (i in 100 downTo 1 step 2) {
    print(i) // 100\n 98\n ...
}
```

* 자바와 같이 반만 닫힌 범위에 대해 이터레이션하려면 `until` 함수를 사용하면 된다.

```kotlin
for (x in 0 until size) { // for (x in 0..size-1) 과 동일하다.
    print(x)
}
```

* Map 타입에 대한 이터레이션은 아래와 같다.

```kotlin
val binaryReps = TreeMap<Char, String>()
binaryReps['A'] = "Apple"
binaryReps['B'] = "Banana"

for ((letter, binary) in binaryReps) {
    print("$letter = $binary")
}
```

* in 연산자를 사용해 특정 값이 범위에 속하는지 검사할 수 있다. 범위 검사는 Comparable 인터페이스를 구현한 모든 클래스에 대해 가능하다.

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'

print(isLetter('q')) // true
print(isLetter('3')) // false
```

* in 연산자를 통해 문자열이 범위 내에 존재하는지 확인할 수 있으며, 컬렉션에도 in 연산을 사용할 수 있다.

```kotlin
print("Kotlin" in "Java".."Scala") // true

print("Kotlin" in setOf("Java", "Scala")) // false
```

## 예외 처리

* 코틀린의 throw, try는 식이기 때문에 다른 식에 포함될 수 있다.

```kotlin
val percentage =
    if (number in 0..100)
        number
    else
        throw IllegalArgumentException("number should between 0 and 100 : $number")
```

```kotlin
fun method() {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException {
        return // 메서드를 종료한다.
    }
}
```

* try-catch-finally 절을 사용할 수 있다.

```kotlin
try {
    val line = reader.readline()
    return Integer.parseInt(line)
} catch (e: NumberFormatException) { // unchecked exception이며, 예외를 바깥으로 전달하지 않고 내부에서 처리
    return null
} finally {
    reader.close()
}
```

* 자바에서는 Checked Exception을 던질 경우 반드시 메서드 시그니처에 `throws <예외타입>`을 명시해주어야 한다. 코틀린에서는 Checked Exception과 Unchecked Exception을 구분하지 않고 모두 Unchecked Exception으로 간주한다.
