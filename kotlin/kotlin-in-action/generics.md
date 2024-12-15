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
* 어떤 타입 A의 값이 필요한 모든 곳에 타입 B의 값을 대입해도 된다면, **타입 B는 타입 A의 하위 타입**이다.
  * 예를 들어 Int는 Number의 하위 타입이지만 String의 하위 타입은 아니다.
* A 타입이 B타입의 하위 타입이라면 B는 A의 상위 타입이다.
* 컴파일러는 변수 대입 또는 함수 인자 전달 시 하위 타입 검사를 수행하여 **변수 타입의 하위 타입인 경우에만 값을 대입**할 수 있도록 한다.
* 대부분의 경우 하위 타입은 하위 클래스와 같다. 하지만 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 타입이지만 반대의 경우는 성립하지 않는 예외 케이스가 있다.

```kotlin
var s: String = "abc"
var t: String? = s // 가능
var ns: String = t // 불가능
```

### 공변성

* **무공변(invariant)**&#xC774;란 인스턴스화할 때 제네릭 타입 인자로 서로 다른 타입이 들어갈 때 하위 타입 관계가 성립하지 않는 경우를 의미한다.
* 자바에서는 모든 클래스가 무공변이다.&#x20;
  * A 클래스가 B 클래스의 하위 타입이더라도, MutableList\<A>는 MubableList\<B>의 하위 타입이 아니다.
* **공변성 클래스**를 선언할 수 있다. 예를 들어 읽기 전용 타입인 List\<A>는 List\<B>의 하위 타입이다. 따라서 List 클래스는 공변적이며 하위 타입 관계가 유지된다.
* 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 앞에 `out`을 붙여야 한다.

```kotlin
interface Producer<out T> {
    fun produce(): T
}
```

* 타입 파라미터 T를 선언한 클래스에서 함수의 **반환 타입에 T를 사용하면 T는 아웃`(out)` 위치**에 있게 되어 T 타입을 생산한다. 함수의 **입력 인자에 T를 사용하면 T는 인`(in)` 위치**에 있게 되어 T 타입을 소비한다.
* 즉, out을 붙이면 하위 타입 관계가 유지되는 공변성을 나타내며, T 타입을 생산만 할 수 있다는 제약이 생긴다. 이를 통해 하위 타입 관계의 타입 안전성을 보장한다.
* val이나 var 키워드를 생성자 파라미터에 적으면 getter/setter를 정의하는 것과 같으므로, val 변수는 out 성격이 되고, var 변수는 in, out 성격을 갖게 된다.
* 생성자의 입력 인자나 private 메서드의 입력 인자는 in도 out도 아니다. 즉, 외부에 노출되어 있는 경우만 in, out을 따질 수 있다.
* 클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않더라도 그 클래스의 인스턴스를 함수 인자나 반환값으로 사용할 수 있다.
* 무공변 클래스를 사용하는 경우 아래와 같이 자동으로 타입 변환이 안되므로 오류가 발생한다. 이를 해결하기 위해서는 코드가 안전한지 따져본 후 명시적 타입 캐스팅을 해주어야 한다.

```kotlin
class Herd<T: Animal> {
    // ...
}

fun feedAll(animals: Herd<Animal>) {
    for (i in 0 until animals.size) {
        animals[i].feed()
    }
}
```

```kotlin
val cats: Herd<Cat> = ...
feedAll(cats) // 타입 변환 불가하여 오류 발생
```

* 무공변 클래스를 공변 클래스로 만들면 명시적 타입 캐스팅 없이도 하위 타입을 입력할 수 있게 된다.

```kotlin
class Herd<out T: Animal> { // out을 붙여 공변 클래스로 만들기
    // ...
}

fun feedAll(animals: Herd<Animal>) {
    for (i in 0 until animals.size) {
        animals[i].feed()
    }
}
```

```kotlin
val cats: Herd<Cat> = ...
feedAll(cats) // 성공
```

### 반공변성

* T 타입에 in 키워드를 붙여야 한다.
* 타입 B가 타입 A의 하위 타입일 때 Consumer\<A>가 Consumer\<B>의 하위 타입이면 Consumer\<T> 클래스는 타입 인자 T에 대해 반공변이다.
* 예를 들어 Consumer\<Animal>은 Consumer\<Cat>의 하위 타입이다.

<figure><img src="../../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

* 즉, Cat 타입을 검증하기 위해 Animal 타입을 검증하는 클래스를 그대로 사용할 수 있다는 의미이다.
* 클래스나 인터페이스의 각 파라미터마다 공변/반공변이 적용될 수 있다.
* 아래는 P 타입은 in 위치, R 타입은 out 위치에서만 사용되도록 정의한 함수 인터페이스이다.

```kotlin
interface Function<in P, out R> {
    operator fun invoke(p: P): R
}
```

* 다음 예시는 공변과 반공변이 함께 있는 경우를 다룬다. Cat을 입력받아 Number를 반환하는 람다에 Animal을 입력받아 Int를 반환하는 함수를 넣을 수 있다. 즉, 상위 타입인 Animal을 입력받고 하위 타입인 Int를 반환하는 함수가 enumerateCats 함수 인자로 입력된다.

```kotlin
fun enumerateCats(f: (Cat) -> Number) {...}
fun Animal.getIndex(): Int = ...

>> enumerateCats(Animal::getIndex)
```

### 사용 지점 변성

* 클래스를 선언하면서 변성을 지정하는 **선언 지점 변성 방식**을 사용하면 해당 클래스를 사용하는 모든 장소에 변성 지정자가 영향을 끼치므로 편리하다.
* 자바에서는 제네릭 타입을 사용할 때 마다 해당 타입 파라미터를 어떤 타입으로 대치할 수 있는지 명시해야 하며 이를 **사용 지점 변성**이라고 부른다.
* 코틀린에서는 기본적으로 선언 지점 변성 방식을 사용하지만, 사용 지점 변성 방식도 제공한다.
* MutableList 같은 상당 수의 인터페이스는 타입 파라미터로 지정된 타입을 소비하는 동시에 생산할 수 있으므로 일반적으로 공변적이지도 반공변적이지도 않다.
* 원본 컬렉션을 복제하는 함수의 경우 원본 컬렉션의 원소를 읽어 새로운 컬렉션에 원소를 쓰게 된다. 이 때 원본 컬렉션의 원소 타입이 새로운 컬렉션의 원소 타입의 하위 타입이어도 동작하도록 아래와 같이 제네릭 타입 두 개를 두어 사용할 수 있다.

```kotlin
fun <T: R, R> copyData(source: MutableList<T>, destination: MutableList<R>) {
    for (item in source) {
        destination.add(item)
    }
}
```

* 위 함수를 out 변성 변경자를 사용해 하나의 제네릭 타입으로도 표현할 수 있다.

```kotlin
fun <T copyData(source: MutableList<out T>, destination: MutableList<R>) {
    for (item in source) {
        destination.add(item)
    }
}
```

* in이나 out 변경자를 붙이면 **타입 프로젝션**이 일어난다. 즉, 원래 타입을 그대로 가져오는 것이 아니라 out이라면 T 타입을 반환하는 메서드만 호출할 수 있는 타입이 되고, in 이라면 T 타입을 인자로 받는 메서드만 호출할 수 있는 타입이 된다.

### 스타 프로젝션

* 제네릭 타입 인자 정보가 없음을 표현할 때 사용한다.
* 예를 들어 원소 타입이 정해지지 않은 리스트는 `List<*>` 로 표현한다.
* 그렇다고 `MutableList<*>`가 `MutableList<Any?>`와 같은 것은 아니다. 스타 프로젝션은 구체적인 한 타입을 저장함을 나타내는 것이므로 아무 원소나 다 담을 수 없다.
* 스타 프로젝션은 자바의 와일드 카드 `<?>`와 대응된다.
* 타입 파라미터를 시그니처에서 언급하지 않거나, 데이터를 읽지만 타입에 관심이 없거나, 타입 인자 정보가 중요하지 않을 때 스타 프로젝션을 사용할 수 있다.

```kotlin
fun printFirst(list: List<*>) {
    if (list.isNotEmpty()) {
        println(list.first()) // first()는 Any? 타입을 반환한다.
    }
}
```

* 위 함수를 제네릭을 사용해 나타내면 아래와 같으며, 제네릭 정보를 알 필요 없을 때 위와 같이 작성하면 된다.

```kotlin
fun <T> printFirst(list: List<T>) {
    if (list.isNotEmpty()) {
        println(list.first()) // first()는 T 타입을 반환한다.
    }
}
```

* **제네릭 타입**을 키로 하고 **제네릭 타입을 사용하는 클래스**를 값으로 하는 맵은 아래와 같이 구현해야 한다.
  * 사용자가 항상 같은 제네릭 타입에 대한 키-값을 입력하도록 하고, 값을 반환 할 때에는 스타 프로젝션 대신 구체적인 제네릭 타입을 지정해 반환해주어야 한다.
  * `FieldValidator<*>` 타입의 객체를 반환받으면 실제로 String, Int에 대한 검증이 불가능하다. 왜냐하면 해당 타입이 어떤 타입을 검증하는지 컴파일러가 모르기 때문이다. 따라서 Validators 클래스에서 구체적인 타입으로 캐스팅 후 반환해주어야 한다.

```kotlin
interface FieldValidator<in T> {
    fun validate(input: T): Boolean
}

object DefaultStringValidator : FieldValidator<String> {
    override fun validate(input: String) = input.isNotEmpty()
}

object DefaultIntValidator : FieldValidator<Int> {
    override fun validate(input: Int) = input >= 0
}

object Validators {
    private val validators =
            mutableMapOf<KClass<*>, FieldValidator<*>>()

    // validator 추가
    fun <T: Any> registerValidator(
            kClass: KClass<T>, fieldValidator: FieldValidator<T>) {
        validators[kClass] = fieldValidator
    }

    // 원하는 타입에 맞는 validator 조회
    @Suppress("UNCHECKED_CAST")
    operator fun <T: Any> get(kClass: KClass<T>): FieldValidator<T> =
        validators[kClass] as? FieldValidator<T>
                ?: throw IllegalArgumentException(
                "No validator for ${kClass.simpleName}")
}
```
