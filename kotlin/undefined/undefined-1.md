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

## enum

* enum은 소프트 키워드로, enum class 와 같이 정의할 수 있으며 enum이라는 문자 자체는 변수명으로 사용이 가능하다.
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





## 프로퍼티



## 제어 구조



## 스마트 캐스트



## 예외 처리

