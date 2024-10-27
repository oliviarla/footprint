# 제네릭스

## 제네릭 타입 파라미터

* 코틀린에서는 제네릭 타입의 타입 인자를 프로그래머가 명시하거나 컴파일러가 추론할 수 있도록 해야 한다.
* 빈 리스트 생성 시 타입 추론이 불가능하므로 아래와 같이 직접 타입 인자를 정해주어야 한다.

```kotlin
val readers: MutableList<String> = mutableListOf()
val readers = mutableListOf<String> ()
```

* 리스트 생성 시 원소를 같이 입력하는 경우 타입 추론이 가능하므로 아래와 같이 간단히 작성할 수 있다.

```kotlin
val authors = listOf("a", "b")
```

* 확장 함수에서 수신 객체나 파라미터 타입에 타입 인자를 사용할 수 있다. 확장 함수를 호출할 때에도 타입 추론이 가능하다.

```kotlin
fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>
```

```kotlin
val authors = listOf("a", "b")
val readers = mutableListOf<String>("a", "b", "c")
readers.filter { it !in authors }
```

* 제네릭 확장 프로퍼티도 선언 가능하다.

```kotlin
val <T> List<T>.penultimate: T
    get() = this[size - 2]
```

```kotlin
val result = listOf(1, 2, 3, 4).penultimate // 3
```

* 제네릭 클래스를 선언할 수 있다.

```kotlin
interface List<T> {
    operator fun get(index: Int): T
}
```

* 제네릭 클래스를 확장하거나 제네릭 인터페이스를 구현하는 클래스를 정의하려면 기반 타입 제네릭 파라미터에 구체적인 타입을 넘기거나 타입 파라미터를 그대로 넘길 수 있다.

```kotlin
class StringList: List<String> {
    override fun get(index: int): String = ...
}

class ArrayList<T>: List<T> {
    override fun get(index: int): T = ...
}
```

### 타입 파라미터 제약

* 타입 파라미터 제약이란 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다.
* 리스트에 속한 모든 원소의 합을 구하는 sum 함수를 Int 리스트나 Double 리스트에만 적용할 수 있도록 하기 위해 이 기능을 사용할 수 있다.
* 타입 파라미터 뒤에 상한을 지정하여 타입 인자가 반드시 상한 타입이거나 상한 타입의 하위 타입이도록 강제할 수 있다. 상한 타입에 정의된 메서드는 T 타입 값을 통해 호출할 수 있다.
* 아래는 Number 타입을 상한으로 지정한 sum 함수의 시그니처이다.

```kotlin
fun <T: Number> List<T>.sum(): T
```

* 파라미터에 대해 둘 이상의 제약을 가해야 하는 경우 where 구문을 사용할 수 있다.
* 아래는 타입 파라미터가 CharSequence, Appendable 두 인터페이스를 구현해야만 한다는 제약을 표현하는 예제이다.

```kotlin
fun <T> ensureTrailingPeriod(seq: T)
    where T : CharSequence, T : Appendable {
    if (!seq.endsWith('.')) {
        seq.append('.')
    }
}
```

* 아무런 상한을 정하지 않은 타입 파라미터는 Any?를 상한으로 한 파라미터가 된다.
* 따라서 안전하게 사용하고자 한다면 물음표를 붙여 안전하게 호출해야 한다.

```kotlin
class Processor<T> {
    fun process(value: T) {
        value?.hashCode()
    }
}
```

* 타입 파라미터를 널이 될 수 없는 타입으로 한정하려면 Any를 상한으로 두어야 한다. 혹은 널이 될 수 없는 타입을 상한으로 두어도 된다.

```kotlin
class Processor<T: Any> {
    fun process(value: T) {
        value.hashcode()
    }
}
```

## 소거된/실체화된 타입 파라미터

* JVM의 제네릭스는 보통 타입 소거를 사용해 구현된다. 따라서 실행 시점에 제네릭 클래스의 객체에 타입 인자 정보가 들어있지 않게 된다.
* 코틀린도 마찬가지로 제네릭 타입 인자 정보가 런타임에 제거된다. 따라서 `List<String>`과 `List<Int>`는 컴파일러 시점에서는 서로 다른 타입이지만 실행 시점에는 같은 타입이 된다.
* 타입 소거로 인해 실행 시점에 타입 인자 검사를 할 수 없다. 따라서 타입 인자로 지정한 타입을 is 로 검사할 수 없다.
* 코틀린에서는 스타 프로젝션을 통해 인자를 알 수 없는 제네릭 타입을 표현한다.

```kotlin
if (value is List<*>) { ... }
```

* 클래스 타입이 같지만 타입 파라미터가 다른 타입으로 캐스팅할 경우, 타입 인자를 알 수 없는 상황에서 컴파일러가 unchecked cast 경고를 발생시킨다.

```kotlin
fun printSum(c: Collection<*>) {
    val intList = c as? List<Int>
        ?: throw IllegalArgumentException("List is expected")
    println(intList.sum())
}
```

### 실체화한 타입 파라미터를 사용한 함수 선언

* 인라인 함수의 타입 파라미터는 실체화되므로 실행 시점에 인라인 함수의 타입 인자를 알 수 있다.&#x20;
* 컴파일러는 인라인 함수의 본문을 인라인 함수를 호출하는 지점마다 삽입하기 때문에 정확한 타입 인자를 알 수 있기 때문이다.
* 아래와 같이 인라인 함수의 타입 파라미터를 reified로 지정하여 실행 시점에 타입 파라미터가 지워지지 않음을 표시하면, 입력된 인자의 타입이 T 타입인지 실행 시점에 검사할 수 있다.

```kotlin
inline fun <reified T> isA(value: Any) = value is T
```

```kotlin
val result = isA<String>("abc") // true
```

* 표준 라이브러리 함수인 filterIsInstance는 인자로 받은 컬렉션의 원소 중 타입 인자로 지정한 클래스의 객체만 모아 리스트로 반환한다.

```kotlin
val items = listOf("one", 2, "three", 4)
val filteredItems = items.filterIsInstance<String>() // [one, three]
```

* 여기서 **인라인 함수를 사용하는 이유**는 람다 파라미터에 해당하는 인자를 함께 인라이닝하여 성능 상 이점을 얻기 위함이 아니라 **실체화한 타입 파라미터를 사용하기 위함**이다. 인라인 함수의 크기가 커진다면 실체화된 타입에 의존하지 않는 부분만 일반 함수로 분리해야 한다.
* 자바 코드에서는  reified 타입 파라미터를 사용하는 inline 함수를 호출할 수 없다. 자바에서는 코틀린 인라인 함수를 일반 함수처럼 호출하기 때문이다.

### 실체화한 타입 파라미터로 클래스 참조

* JDK의 ServiceLoader에서는 추상 클래스나 인터페이스를 표현하는 Class 객체를 입력받아 해당 클래스나 인스턴스를 구현한 인스턴스를 반환한다.

```kotlin
val serviceImpl = ServiceLoader.load(Service::class.java)
```

* 구체화한 타입 파라미터를 사용해 인라인 함수를 정의하면 아래와 같이 간결하게 사용할 수 있다.

```kotlin
inline fun <reified T> loadService() {
    return ServiceLoader.load(T::Class.java)
}
```

```kotlin
val serviceImpl = loadService<Service>()
```

### 실체화한 타입 파라미터의 제약

* reified 타입 파라미터로 가능한 작업은 다음과 같다.
  * 타입 검사와 캐스팅 (is,!is,as,as?)
  * 10장에서 설명할 코틀린 리플렉션 API(::class)
  * 코틀린 타입에 대응하는 java.lang.Class 얻기 (::class.java)
  * 다른 함수를 호출할 때 타입 인자로 사용
* reified 타입 파라미터로 불가능한 작업은 다음과 같다.
  * 타입 파라미터 클래스의 인스턴스 생성하기
  * 타입 파라미터 클래스의 동반 객체 메소드 호출하기
  * reified 타입 파라미터를 요구하는 함수를 호출하면서 reified 하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
  * 클래스, 프로퍼티, 인라인 함수가 아닌 일반 함수의 타입 파라미터를 reified로 지정하기

## 변성: 제네릭과 하위 타입

### 필요성

* 원소의 추가나 변경이 있다면 List\<Any>를 인자로 받는 함수에 List\<String>을 입력할 수 없다.
* 아래와 같이 addAnswer에서 list.add가 호출되는 순간 List\<String>에 Int를 입력하게 되어ClassCastException이 발생하게 된다.

```kotlin
fun addAnswer(list: MutableList<Any>) {
    list.add(42)
}

val strings = mutableListOf("abc","bac")
addAnswer(strings)
println(strings.maxBy { it.length })
```

### 클래스, 타입, 하위 타입

* 제네릭이 아닌 클래스 이름은 바로 타입으로 쓸 수 있다.
* 클래스 이름을 널이 될 수 있는 타입에도 쓸 수 있으므로 모든 코틀린 클래스가 적어도 둘 이상의 타입을 구성할 수 있다.

```kotlin
var x: String
var x: String?
```

* 각각의 제네릭 클래스는 무수히 많은 타입을 만들 수 있다. 예를 들어 List는 클래스이고, List\<String?>, List\<Int> 는 타입이다.
* 어떤 타입 A의 값이 필요한 모든 곳에 타입 B의 값을 대입해도 된다면, 타입 B는 타입 A의 하위 타입이다.









