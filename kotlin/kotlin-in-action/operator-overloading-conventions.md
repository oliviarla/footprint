# 연산자 오버로딩과 기타 관례



> 코틀린에서 관례란 **어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법**을 의미한다.

## 산술 연산자 오버로딩

* 자바에서는 String, 원시 타입에 대해서만 산술 연산자인 `+`를 사용할 수 있다.
* 코틀린에서는 BigInteger 클래스에서 두 값을 더하거나 컬렉션에 원소를 추가하는 경우에도 산술 연산자를 사용할 수 있도록 해준다.
* 코틀린 언어 자체에서 미리 정해둔 연산자만 오버로딩 가능하며, 관례를 따르기 위해 클래스에서 정의해야 하는 이름이 정해져 있다.

### 이항 산술 연산 오버로딩

* 각각의 이항 산술 연산 식에 대해 정해진 함수 이름은 다음과 같다.
* 각 연산자의 우선 순위는 표준 숫자 타입에 대한 우선 순위와 같다. 따라서 \*, /, % 연산이 우선이고, +, -는 그 다음에 수행된다.

| 식      | 함수 이름            |
| ------ | ---------------- |
| a \* b | times            |
| a / b  | div              |
| a % b  | mod (1.1 부터 rem) |
| a + b  | plus             |
| a - b  | minus            |

* operator 키워드를 단 plus 함수를 클래스에 정의하면 관례를 따르는 함수가 되며, `+` 기호를 사용해 두 객체를 더한 결과를 얻을 수 있다.

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}
```

* 연산자를 확장 함수로 정의할 수도 있다.
* 외부 함수의 클래스에 대한 연산자를 정의할 때에는 확장 함수로 관례를 따르는 함수를 정의하는 것이 일반적이다.

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```

* 자바 클래스에 코틀린의 관례에 맞아 떨어지는 메서드가 구현되어 있다면 코틀린에서 연산자를 사용할 수 있다. operator 변경자는 자바에 없으므로 이름과 파라미터 개수만 일치하면 된다. 만약 원하는 연산 기능을 제공하는 메서드가 제공되고 있지만 관례에 맞지 않다면, 확장 함수를 정의하고 내부적으로 해당 메서드를 호출하면 된다.
* 연산자 정의 시 피연산자들이 반드시 같은 타입일 필요는 없다.
* 아래는 Point 타입에 Double 타입으로 `*` 할 수 있도록 하는 연산자 함수의 예제이다.

```kotlin
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}
```

* 연산자 함수의 반환 타입이 피연산자 타입들 중 하나와 일치할 필요도 없다.

```kotlin
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}
```

* 코틀린 연산자가 자동으로 교환 법칙 (`a op b == b op a`)을 지원하지 않으므로, 만약 필요하다면 양쪽 타입에 연산자 함수를 정의해주어야 한다.

```kotlin
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}

operator fun Double.times(point: Point): Point {
    return Point((point.x * this).toInt(), (point.y * this).toInt())
}
```

* operator 함수도 오버로딩이 가능하여 다양한 파라미터 타입을 입력받도록 지원할 수 있다.

### 복합 대입 연산자 오버로딩

* `+=` , `-=` 등의 연산자를 복합 대입 연산자라고 한다.
* 아래와 같이 복합 대입 연산자를 통해 객체의 내부 상태를 변경하기 위해서는 복합 대입 연산자 함수를 정의해야 한다.

```kotlin
var point = Point(1, 2)
point += Point(3, 4)
```

* 반환 타입이 Unit인 plusAssign 함수를 정의하면 += 연산자를 사용 시 해당 함수가 사용된다. minusAssign, timesAssign 함수도 마찬가지로 `-=` , `*=` 연산을 지원하기 위해 정의할 수 있다.
* 코틀린 표준 라이브러리는 변경 가능한 MutableCollection 인터페이스에 대해 plusAssign 확장 함수를 정의해두고 있다.

```kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}
```

* `a += b`는 `a = a.plus(b)` 혹은 `a.plusAssign(b)` 로 컴파일될 수 있다. 따라서 변경 불가능한 클래스라면 plus 함수만 추가하고, 변경 가능한 클래스라면 plusAssign 함수만 추가해야 컴파일 오류가 나지 않는다.
* 코틀린 표준 라이브러리가 제공하는 연산자 특징은 다음과 같다.
  * \+, - 연산자는 항상 새로운 컬렉션을 반환한다.
  * \+=, -= 연산자는 변경 가능한 컬렉션에 작용해 객체의 상태를 변화시킨다.
  * 읽기 전용 컬렉션에서는 +=, -= 연산 시 변경을 적용한 복사본을 반환한다.

### 단항 연산자 오버로딩

* `-a`, `+a` 같은 단항 연산자도 오버로딩할 수 있다. 아래는 각 단항 연산자에 매핑되는 함수 이름 목록이다.

| 식        | 함수 이름      |
| -------- | ---------- |
| +a       | unaryPlus  |
| -a       | unaryMinus |
| !a       | not        |
| ++a, a++ | inc        |
| --a, a-- | dec        |

* 아래와 같이 감소 연산자를 정의할 수 있다.

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}
```

* inc/dec 함수를 오버로딩하는 경우 일반적인 값에 대한 전위, 후위 증가/감수 연산자와 같은 의미를 제공한다. ++a는 전위 증가 연산자이므로 값을 증가시킨 후 a값을 사용하게 되고, a++는 후위 증가 연산자이므로 a값을 먼저 사용한 후 증가시킨다.

## 비교 연산자 오버로딩

### equals

* 코틀린은 == 연산자를 사용한 코드를 equals 메서드를 호출하도록 컴파일한다.
* a == b 이라는 코드는 아래와 같이 a가 null이 아닌 경우에만 equals를 호출하고, 만약 null이면 b도 null인지 확인하여 결과를 반환한다.

```kotlin
a?.equals(b) ?: (b == null)
```

* 코틀린에서는 식별자 연산자 `===` 를 통해 두 객체가 동일한지 확인할 수 있다. 따라서 `===`는 오버라이드할 수 없다.
* 다른 연산자 오버로딩 관례와 달리 equals는 Any 클래스에 이미 정의되어 있으므로 오버라이드하기 위해 override를 붙여야 한다.
* Any 클래스에 equals가 정의되어 있으며 operator 변경자가 붙어있는데, 오버라이드할 때에는 상위 클래스의 operator가 적용되므로 따로 변경자를 붙이지 않아도 된다.

```kotlin
class Point(val x: Int, val y: Int) {
    override fun equals(obj: Any?): Boolean {
        if (obj === this) return true
        if (obj !is Point) return false
        return obj.x == x && obj.y == y
    }
}
```

### compareTo

* 자바에서는 정렬, 최댓값, 최솟값 등을 구하기 위해 값을 비교할 때 Comparable 인터페이스를 구현해야 한다. 이 때 항상 compareTo 메서드를 통해 객체를 비교해야 한다.
* 코틀린에서는 compareTo 메서드를 호출하는 관례를 제공하여 비교 연산자(`<` `>` `<=` `>=` )를 코드에서 사용하면 compareTo 메서드가 호출되도록 한다.
* p1 < p2 라고 코드를 작성하면 p1.compareTo(p2) < 0 으로 동작하게 된다.
* 아래는 Person 클래스가 Comparable 인터페이스를 구현하는 예제이다. 코틀린에서 제공하는 compareValuesBy 함수는 두 객체와 비교 함수(람다 혹은 메서드/프로퍼티 참조)들을 인자로 받아 순차적으로 비교 함수를 적용하며 결과를 반환한다.

```kotlin
class Person(
        val firstName: String, val lastName: String
) : Comparable<Person> {

    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other,
            Person::lastName, Person::firstName)
    }
}
```

* 자바 클래스가 Comparable 인터페이스를 구현하고 있다면 코틀린에서는 해당 객체들에 대해 비교 연산자를 사용할 수 있다.

## 컬렉션과 범위에 대해 쓸 수 있는 관례

* 코틀린에서는 인덱스를 사용해 컬렉션의 원소를 가져올 때 `list[idx]` 식을 사용하거나, 컬렉션에 어떤 값이 속해있는지 확인할 때 `item in list` 식을 사용할 수 있다. 사용자 지정 클래스에서도 이러한 식을 지원할 수 있다.

### get / set

* 인덱스 연산자를 사용해 원소를 읽는 연산은 get 함수를 호출하고, 원소를 쓰는 연산은 set 함수를 호출한다.
* 점의 x, y 값을 point\[0], point\[1] 과 같이 접근하고자 한다면 아래와 같이 get 함수를 정의하면 된다.

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("invalid coordinate $index")
    }
}
```

* 인덱스로 주어지는 값의 타입은 Int 외에도 다양한 타입이 될 수 있다.
* 2차원 행렬이나 배열을 표현하는 클래스에서 array\[row, col]과 같은 형태로 접근하려면 `operator fun get(rowIndex: Int, colIndex: Int)` 함수를 정의해야 한다.
* 인덱스에 해당하는 원소를 `point[1] = 4`와 같은 형태로 쓰고 싶다면 set 함수를 정의하면 된다. 함수의 마지막 파라미터 값은 대입문의 우항에 들어가고 나머지 파라미터들은 인덱스 연산자에 들어가게 된다.

```kotlin
data class MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
    when(index) {
        0 -> x = value
        1 -> y = value
        else -> hrow IndexOutOfBoundsException("invalid coordinate $index")
    }
}
```

### in

* 객체가 컬렉션에 존재하는지 확인하는 in 연산자를 사용하려면 contains 함수를 정의하면 된다.
* 아래는 사각형 객체의 영역에 점이 존재하는지 `point in rectangle` 식으로 확인하기 위해 contains 확장 함수를 정의하는 예제이다.
  * until 함수를 사용해 끝값을 포함하지 않는 열린 범위를 만들 수 있다. 닫힌 범위를 만드려면 `10..20` 처럼 `..` 를 사용하면 된다.

```kotlin
data class Point(val x: Int, val y: Int)

data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x until lowerRight.x &&
           p.y in upperLeft.y until lowerRight.y
}
```

### rangeTo

* `..` 연산자는 rangeTo 함수를 호출한다. `10..20` 식을 사용하면 10부터 20까지의 범위를 나타내게 된다.
* rangeTo 함수는 범위를 반환하며, Comparable 인터페이스가 구현되어 있는 클래스라면 따로 정의할 필요가 없다. 이미 코틀린 표준 라이브러리에서 정의되어 있기 때문이다.

```kotlin
operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

* 다음은 현재 날짜부터 10일동안 방학이고, 그 사이에 다음주 날짜가 해당되는지 확인하는 예제이다.

```kotlin
val now = LocalDate.now()
val vacation = now..now.plusDays(10)
val inVacation = now.plusWeeks(1) in vacation
```

* 범위 연산자는 우선 순위가 낮기 때문에 범위에 대한 forEach 반복문을 사용하려면 괄호로 둘러싼 후 사용해야 한다.

```kotlin
(0..n).forEach { print(it) }
```

### iterator

* for 루프 내에서 in 연산자를 사용하면 내부적으로 iterator 함수를 호출해 Iterator를 얻고 hasNext, next를 호출한다.
* iterator 함수를 통해 Iterator 객체를 반환하면 된다.
* 코틀린 표준 라이브러리는 String의 상위 클래스인 CharSequence에 대한 iterator 확장 함수를 제공한다.

```kotlin
operator fun CharSequence.iterator(): CharIterator
```

* 다음은 날짜 범위인 ClosedRange\<LocalDate> 타입에 대해 iterator 확장 함수를 제공하여 forEach에서 in 연산자를 사용할 수 있도록 하는 예제이다.

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    object : Iterator<LocalDate> {
        var current = start
    
        override fun hasNext() =
            current <= endInclusive
    
        override fun next() = current.apply {
            current = plusDays(1)
        }
    }
}
```

```kotlin
val newYear = LocalDate.ofYearDay(2025, 1)
val daysOff = newYear.minusDays(1)..newYear
for (dayOff in daysOff) { println(dayOff) }
```

## 구조 분해 선언과 component 함수

### 구조 분해 선언

* 구조 분해 선언이란 복합적인 값을 분해해 다른 변수들을 한꺼번에 초기화하는 것이다.
* 아래와 같이 p 변수의 컴포넌트들을 이용해 x, y 변수를 초기화할 수 있다.

```kotlin
val p = Point(10, 20)
val (x, y) = p
```

* 내부적으로는 아래와 같이 componentN 함수를 호출하여 값을 할당한다.

```kotlin
val x = p.component1()
val y = p.component2()
```

* data 클래스에서는 주 생성자에 들어있는 프로퍼티에 대해 컴파일러가 자동으로 componentN 함수를 만들어준다.
* 사용자 정의 클래스에서는 직접 componentN 함수를 구현해주어야 한다.

```kotlin
class Point(x: Int, y: Int) {
    operator fun component1() = x
    operator fun component2() = y
}
```

* 구조 분해 선언은 함수에서 여러 값을 반환해야 할 때 값들이 담긴 데이터 클래스를 반환하고, 해당 클래스의 값들을 각각의 변수에 바로 할당할 수 있다.
* 다음은 String으로 된 파일 이름을 입력받아 `.`을 기준으로 분리해 NameComponents라는 데이터 클래스를 반환하고, 구조 분해를 통해 각각의 변수에 저장하는 예제이다.

```kotlin
data class NameComponents(val name: String,
                          val extension: String)

fun splitFilename(fullName: String): NameComponents {
    val result = fullName.split('.', limit = 2)
    return NameComponents(result[0], result[1])
}

```

```kotlin
val (name, ext) = splitFilename("example.kt")
```

* 크기가 정해진 컬렉션에 대해서도 구조 분해가 가능하다. 코틀린 표준 라이브러리에서는 맨 앞의 다섯 원소에 대한 componentN 함수를 제공한다. 만약 6번째 원소를 구조 분해하고자 한다거나 컬렉션의 범위를 넘어서는 만큼 구조 분해를 하려고 하면 예외가 발생한다.
* 아래는 String을 `.` 을 기준으로 분리해 생성된 컬렉션 원소를 구조 분해 선언으로 할당하는 예제이다.

```kotlin
val fullname = "hello.kt"
val (name, extension) = fullname.split('.', limit = 2)
```

### 구조 분해 선언과 루프

* 루프 안에서 구조 분해 선언을 사용할 수 있다.
* 아래는 맵의 원소에 대해 이터레이션할 때 모든 원소를 출력할 때 구조 분해 선언을 사용하는 예제이다.
  * 코틀린에서는 Map.Entry에 대해 component1, component2 확장 함수를 제공하여 구조 분해 선언에서 사용할 수 있도록 한다.

```kotlin
val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")

for ((key, value) in map) {
    println("$key -> $value")
}
```

## 위임 프로퍼티

### 위임 프로퍼티

* 위임이란 객체가 직접 작업을 수행하는 대신, 작업을 처리하는 다른 도우미 객체(위임 객체)에게 맡기는 디자인 패턴이다.
* 값을 필드가 아닌 데이터베이스 테이블이나 브라우저 세션 등 복잡한 방식으로 작동하는 프로퍼티에 저장하도록 구현할 수 있다.
* 일반적인 문법은 아래와 같다.
  * p 프로퍼티는 접근자 로직을 다른 객체에게 위임한다.
  * by 뒤에 있는 식을 통해 위임 객체를 얻는다.

```kotlin
class Foo {
    var p: Type by Delegate()
}
```

* 컴파일러는 숨겨진 도우미 프로퍼티인 delegate를 만들어 위임 객체의 인스턴스로 초기화한다. p 프로퍼티는 위임 객체에게 자신의 작업을 위임하게 된다.

```kotlin
class Foo {
    private val delegate = Delegate()
    var p: Type
    set(value: Type) = delegate.setValue(..., value)
    get() = delegate.getValue(...)
}
```

* Delegate 클래스는 getValue, setValue 메서드를 멤버 메서드 혹은 확장 함수로 제공해야 한다. setValue의 경우 변경 가능한 프로퍼티에만 구현하면 된다.

```kotlin
class Delegate {
    operator fun getValue(...) {...}
    operator fun setValue(..., value: Type) {...}
}
```

* 아래와 같이 Foo 클래스의 p 프로퍼티에 접근할 때 위임 객체에 의해 접근된다.

```kotlin
val foo = Foo()
val value = foo.p // Delegate의 getValue 호출
foo.p = newvalue // Delegate의 setValue 호출 
```

### by lazy()로 프로퍼티 초기화 지연

* 지연 초기화는 객체를 생성할 때 모두 초기화하지 않고 일부를 남겨두었다가 실제 값이 필요할 때 초기화하는 방법이다.
* 초기화 과정에서 자원을 많이 사용하거나, 객체를 사용할 때 마다 반드시 초기화하지 않아도 되는 프로퍼티가 있는 경우 유용하다.









