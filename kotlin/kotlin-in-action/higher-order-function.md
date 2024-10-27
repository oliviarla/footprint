# 고차 함수

## 고차 함수

* 고차 함수란 다른 함수를 인자로 받거나 함수를 반환하는 함수이다.
* 코틀린에서는 람다나 함수 참조를 인자로 넘기거나 반환하면 고차 함수가 된다.

### 함수 타입

* 람다를 로컬 변수에 대입할 때 코틀린의 타입 추론으로 인해 변수 타입을 지정하지 않아도 된다.
* 이 때 컴파일러는 람다가 함수 타입임을 추론하게 된다.
* 아래는 자동으로 타입 추론이 되도록 간결하게 작성할 수도 있고 직접 함수 타입을 명시할 수도 있음을 보여주는 예제이다.

```kotlin
// 자동으로 타입 추론이 되도록 한다.
val sum = {x: Int, y: Int -> x + y }

// 직접 함수 타입을 명시한다.
val sum: (Int, Int) -> Int = { x, y -> x + y}
```

* 함수 타입을 정의하려면 함수 파라미터의 타입을 괄호 안에 넣고 화살표를 추가한 후 함수의 반환 타입을 지정하면 된다.
* 함수 타입 선언 시 반환값이 없다면 Unit 타입을 반환하도록 지정해야 한다.

```kotlin
(Int, String) -> Unit
```

* 반환 타입을 널이 될 수 있는 타입으로 지정할 수 있다.

```kotlin
var canReturnNull: (Int, Int) -> Int? = { null }
```

* 함수 타입 자체가 널이 될 수 있도록 지정할 수도 있다. 이 경우 함수 타입 전체를 괄호로 감싼 후 물음표를 붙여야 한다.

```kotlin
var funOrNull: ((Int, Int) -> Int)? = null
```

### 인자로 받은 함수 호출

* 인자로 받은 함수는 일반 함수를 호출하는 것과 동일하게 호출하면 된다.
* 아래는 predicate라는 함수 타입을 인자로 받아 문자열에서 특정 조건을 만족하는 문자만 남기는 filter 함수 구현이다.

```kotlin
fun String.filter(predicate: (Char) -> Boolean): String {
    val sb = StringBuilder()
    for (index in 0 until length) {
        val element = get(index)
        if (predicate(element)) sb.append(element)
    }
    return sb.toString()
}
```

```kotlin
val result = "a1b2c3".filter { it in 'a'..'z' } // abc
```

### 자바에서 코틀린의 함수 타입 사용

* 함수 타입 변수는 FunctionN 인터페이스를 구현하는 객체를 저장한다. FunctionN 인터페이스에는 invoke 메서드를 정의해두어 이를 호출하면 함수가 실행된다.
* 코틀린 표준 라이브러리는 함수 인자 개수에 따라 Function0\<R>, Function1\<P1, R> 등의 인터페이스를 제공한다.
* 자바에서는 함수 타입을 인자로 받는 코틀린 함수에 람다를 넘기면 된다. 자바 8 이전이라면 FunctionN 인터페이스의 invoke 메서드를 구현한 익명 클래스를 넘기면 된다.

```kotlin
fun processTheAnswer(f: (Int) -> Int) {
	println(f(42))
}
```

```java
// java 8 이후 람다를 사용한다
processTheAnswer(number -> number + 1)

// java 8 이전에는 익명 클래스를 사용한다
processTheAnswer(
    new Function1<Integer, Integer>() {
        @Override
        public Integer invoke(Integer number) {
            System.out.println(number);
            return number + 1;
        }
    });
```

* 람다를 인자로 받는 확장 함수를 자바에서 호출 시 첫 번째 인자로 수신 객체를 넘겨야 한다.
* 반환 타입이 Unit인 함수나 람다를 자바로 작성할 때 반드시 Unit 타입의 값을 명시적으로 반환해야 한다.
* 아래는 자바로 람다를 인자로 받는 확장 함수에 Unit 타입을 반환하는 람다를 인자로 입력하는 예제이다.

```java
List<String> strings = List.of("42");
CollectionsKt.forEach(strings, s -> {
    System.out.println(s);
    return Unit.INSTANCE;
});
```

### 디폴트 값을 지정하거나 널이 될 수 있는 함수 타입 파라미터

* 함수 타입의 파라미터에 대해 디폴트 값을 지정할 수 있다.
* 다음은 컬렉션의 요소들을 원하는 형태로 가공해 하나의 String 타입으로 통합하는 joinToString 함수이다.&#x20;
  * transform이라는 함수 타입 인자에 대한 기본값을 toString으로 지정하여, 사용자가 별도로 지정하지 않으면 기본값을 사용하게 된다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element))
    }
    result.append(postfix)
    return result.toString()
}
```

* 널이 될 수 있는 함수 타입 파라미터는 물음표를 사용해 선언하고, invoke 메서드 호출 전 물음표를 사용해 null 인지 여부를 확인해야 한다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: ((T) -> String)? = null
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        val str = transform?.invoke(element) ?: element.toString()
        result.append(str)
    }
    result.append(postfix)
    return result.toString()
}
```

### 함수 타입을 반환하기

* 다른 함수를 반환하는 함수를 정의하려면 함수 타입을 반환하도록 해야 한다.

```kotlin
fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}
```

### 람다로 중복 제거하기

* 아래와 같이 사이트 방문 로그 데이터가 존재할 때, 모바일 OS의 사용자들의 평균 사이트 지속 시간을 알고 싶다면 람다를 직접 작성해 filter 함수에 넘겨야 한다.

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)
enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }
val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)
```

```kotlin
val averageMobileDuration = log
    .filter { it.os in setOf(OS.IOS, OS.ANDROID) }
    .map(SiteVisit::duration)
    .average()
```

* 람다를 직접 정의하는 대신 사용자가 직접 원하는 조건이면 true를 반환하는 람다를 함수에 인자로 입력하도록 하여 평균 사이트 지속 시간을 구하도록 할 수 있다.

```kotlin
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate).map(SiteVisit::duration).average()
```

## 인라인 함수

### 동작 방식

* 람다는 보통 익명 클래스로 컴파일되지만 람다 식을 사용할 때마다 새로운 클래스가 생성되지는 않으며, 람다가 변수를 포획하면 람다가 생성되는 시점마다 새로운 익명 클래스 객체가 생성된다.
* 코틀린에서는 반복되는 코드를 별도 함수로 분리할 때 효율적인 동작을 위해 컴파일 시 함수 본문에 바이트코드로 넣어주는 inline 변경자를 제공한다.
* inline 함수를 호출하는 함수 내부 코드에 inline 함수 구현이 바이트코드로 그대로 들어가게 된다.
* 예를 들어 synchronized라는 간단한 함수를 inline으로 정의하면, 컴파일 전에는 함수를 호출하도록 구현해두지만 컴파일하면 별도의 함수를 호출하는 대신 하나의 함수에 흡수(인라이닝)된다.
  * 인자로 입력된 람다 역시 인라이닝된다.

```kotlin
inline fun <T> synchronized(lock: Lock, action: () ->T): T {
    lock.lock()
    try {
        return action()
    } finally {
        lock.unlock()
    }
}
```

```kotlin
// 컴파일 전 코드
fun foo(l: Lock) {
    println("Before sync")
    synchronized(l) {
        println("Action")
    }
    println("After sync")
}

// 컴파일 후 코드
fun __foo__(l: Lock) {
    println("Before sync")
    l.lock()
    try {
        println("Action")
    } finally {
        lock.unlock()
    }
    println("After sync")    
}
```

* 만약 함수 타입의 변수를 인라인 함수에 전달한다면, 컴파일 시점에 변수에 저장된 내용을 알 수 없으므로 함수 타입은 인라이닝되지 않는다.
* 하나의 인라인 함수를 여러 곳에서 호출한다면 각각의 호출된 부분에 인라이닝된다.

### 한계

* 인라인 함수 호출 시 람다 식을 바로 호출하거나 인자로 전달받는 경우에만 인라이닝할 수 있다.
* 인라인 함수의 특정 인자만 인라인되지 않도록 하려면 `noinline` 변경자를 파라미터에 붙여야 한다.

```kotlin
inline fun foo(task1: () -> Unit, noinline task2: () -> Unit) {
    // ...
}
```

### 컬렉션 연산 인라이닝

* 코틀린의 컬렉션에서 제공하는 filter 함수는 인라인 함수이다. 따라서 함수에 전달된 람다 본문과 filter 함수 구현이 filter 함수 호출 위치에 들어간다.
* 아래와 같이 filter, map을 체이닝해 사용하는 경우 호출 위치로 인라인 되지만, 중간 리스트를 생성해 filter 함수의 결과를 저장해두어야 한다.

```kotlin
people.filter { it.age > 30 }
      .map(Person::name)
      // ...
```

* 시퀀스를 사용할 경우 람다를 저장해야 하므로 인라인할 수 없다. 따라서 컬렉션 크기가 클 때에만 시퀀스를 통해 지연 계산으로 성능을 향상시키는 것을 고려해야 한다.

### 함수를 인라인으로 선언해야 하는 이유

* 일반 함수 호출 시 JVM은 코드 실행을 분석해 가장 좋은 방향으로 인라인한다. 이는 JIT 컴파일러가 바이트코드를 기계어 코드로 번역할 때 이뤄진다.
* JVM의 최적화를 사용하면 바이트 코드에서 함수 구현이 한 번만 있으면 되므로 코드가 중복될 필요가 없다.
* 코틀린의 인라인을 사용하면 호출되는 부분마다 코드에 중복이 생기게 되는 단점이 존재하지만, 함수 호출 비용을 줄이고 람다를 표현하는 클래스나 객체를 생성할 필요가 없어진다.
* 인라인 변경자를 붙이는 함수는 코드 크기가 적어야 전체 바이트 코드 크기에 영향이 적으므로 효율적이다.

### 자원 관리를 위해 인라인된 람다 사용

* 파일, 락, 데이터베이스 트랜잭션 등 자원 관리 패턴을 만들 때 인라인된 람다를 사용할 수 있다.
* 다음은 withLock 함수에 락을 사용하는 람다를 입력하여 직접 락을 걸거나 해제하지 않아도 되는 예제이다.

```kotlin
fun <T> Lock.withLock(action: () -> T): T {
    lock()
    try {
        return action()
    } finally {
        unlock()
    }
}
```

```kotlin
val l:Lock = ...
l.withLock(...)
```

* use 함수를 사용하면 자바의 try-with-resource와 같이 직접 자원을 해제해줄 필요 없이 자원을 사용하는 람다만 넘기면 된다.

```kotlin
fun readFirstLineFromFile(path: String): String {
    BufferedReader(FileReader(path)).use { 
        br ‐> return br.readLine()
    }
}
```

## 고차 함수 안에서 흐름 제어

### 람다 안의 return문

* 람다 안에서 return을 사용하면 람다로부터만 반환되는 것이 아니라 람다를 호출하는 함수까지 실행을 끝내고 반환하게 된다.
* 이와 같이 자신을 둘러싸고 있는 블록보다 더 바깥의 블록을 반환하게 만드는 return문을 **non-local return** 이라고 부른다.
* 람다를 인자로 받는 함수가 인라인 함수인 경우에만 non-local return을 할 수 있다. 인라이닝되지 않는 함수는 람다를 변수에 저장할 수 있고, 바깥쪽 함수가 반환된 이후에 람다가 호출될 때를 다룰 수 없으므로 non-local return을 할 수 없다.
* forEach는 인라인 함수이므로 아래와 같이 작성하면 return문을 수행할 때 lookForAlice 함수도 반환된다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}
```

### 레이블을 사용한 return

* local return은 람다의 실행을 끝낸 후 람다를 호출했던 부분을 이어서 실행한다.
* 레이블을 사용해 return으로 실행을 끝내고 싶은 람다식을 지정할 수 있다.
* 람다 식에 레이블을 붙이기 위해 `<레이블 이름>@`을 람다 정의 직전 선언하고, 반환 시에 return문 뒤에 `@<레이블 이름`을 붙인다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach label@ {
        if (it.name == "Alice") return@label
    }
    println("Alice might be somewhere")
}
```

* 람다를 인자로 받는 인라인 함수의 이름을 return문 뒤의 레이블 이름으로 사용할 수도 있다. 람다 식에 레이블을 이미 명시했다면 함수 이름을 레이블로 사용할 수 없다. 람다 식에는 레이블이 2개 이상 붙을 수 없기 때문이다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") return@forEach
    }
    println("Alice might be somewhere")
}
```

### 익명 함수

* 익명 함수는 코드 블럭을 함수에 넘길 때 사용할 수 있는 또다른 방법이다.
* 람다 식에 대한 문법적 편의를 위해 제공된다.
* 일반 함수와 달리 함수 이름이나 파라미터 타입을 생략할 수 있다.
* 별도로 레이블을 붙이지 않아도 return문은 항상 익명 함수 자체만 반환시킨다. 따라서 람다와 달리 바깥 함수가 반환되는 것을 고려하지 않아도 된다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (it.name == "Alice") return
        println("${person.name} is not Alice")
    })
}
```
