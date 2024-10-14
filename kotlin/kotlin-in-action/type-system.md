# 타입 시스템

## 널 가능성

* null이 될 수 있는지 여부를 타입 시스템에 추가하여 컴파일러 단에서 미리 감지해 실행 시점에 발생할 수 있는 예외 가능성을 줄일 수 있다.
* 기본적으로 null 또는 null이 될 수 있는 인자를 넘기지 못하도록 되어있다.

```kotlin
fun strLen(s: String) = s.length
```

* null을 인자로 받을 수 있게 하려면 타입 뒤에 물음표를 명시해야 한다.

```kotlin
fun strLen(s: String?) = s.length
```

* 널이 될 수 있는 값은 널이 될 수 없는 타입에 할당할 수 없다.

```kotlin
val x: String? = null
val y: String = x // ERROR: Type mismatch: inferred type is String? but String was expected
```

* 널이 아님이 확실한 영역에서는 해당 값을 널이 될 수 없는 타입처럼 사용할 수 있다. 예를 들어 아래와 같이 널인지 값을 검사한 후에 사용한다면 널이 될 수 없는 타입처럼 사용할 수 있다.

```kotlin
fun strLen(s: String?) : Int =
    if (s != null) s.length else 0
```

* 타입이란 어떤 값들이 들어올 수 있는지와 타입에 대해 수행할 수 있는 연산의 종류를 결정한다.&#x20;
* String 타입과 double 타입은 각각 할당 가능한 값과 실행할 수 있는 연산이 다르다. 하지만 null 값이 해당 타입에 할당될 경우 일반적인 값과 달리 실행할 수 있는 연산이 없어지게 된다.
* 코틀린에서는 널이 될 수 있는 타입과 널이 될 수 없는 타입을 구분하여 각 타입의 값에 어떤 연산이 가능한 지 쉽게 파악하도록 한다. 또한 이를 컴파일 타임에 검증하게 된다.

### ?. 호출 연산자

* null 검사와 메서드 호출을 한 번에 수행한다.
* 만약 호출 대상 객체가 null이라면 메서드를 수행하는 대신 null을 바로 반환한다.
* 예를 들어 `s?.toUpperCase()` 는 `if (s != null) s.toUpperCase else null` 과 같다. 또한 반환 결과가 널이 될 수 있는 타입인 `String?`이 된다.
* 프로퍼티를 읽거나 쓸 때에도 `?.` 연산자를 사용할 수 있다.

```kotlin
fun managerName(employee: Employee): String? = employee.manager?.name
```

* `?.` 연산자 호출을 연쇄적으로 하여 여러 객체의 null 검사를 쉽게 수행할 수 있다.

```kotlin
fun Person.countryName(): String {
   val country = this.company?.address?.country
   return if (country != null) country else "Unknown"
}
```

### ?: 엘비스 연산자

* null 대신 사용할 디폴트 값을 지정할 때 사용하는 연산자이다.
* 아래는 s가 null이 아니면 그대로 변수에 할당하고, null이면 ""를 변수에 할당하는 예시이다.

```kotlin
val t: String = s ?: ""
```

* 코틀린에서는 return이나 throw 등의 연산도 식이다. 따라서 엘비스 연산자의 우항에 return, throw 등의 연산을 넣을 수 있다.
* 아래는 사람 객체에 연관된 회사의 주소가 없다면 예외를 발생시키는 예시이다.

```kotlin
fun printShippingLabel(person: Person) {
    val address = person.company?.address
      ?: throw IllegalArgumentException("No address")
    with (address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}
```

### as?

* 어떤 값을 지정한 타입으로 캐스트할 수 있다면 변환하고, 불가능하면 null을 반환한다.
* 아래 예제에서는 인자를 Any 타입으로 받았을 때 `as?` 를 통해 타입 캐스팅을 하고, 만약 변환에 실패해 null이 반환되면 `?:` 연산자로 검사한다.

```kotlin
fun equals(o: Any?): Boolean {
       val otherPerson = o as? Person ?: return false
       
       return otherPerson.firstName == firstName &&
              otherPerson.lastName == lastName
}
```

### !! 단언문

* 널이 될 수 있는 타입의 값을 다룰 때 널이 될 수 없는 타입으로 변환하는 연산자이다.
* 널에 대해 `!!`를 적용하면 NPE가 발생한다.
* 아래와 같이 널이 될 수 있는 값에 `!!`를 적용하면 널일 때는 NPE를 발생시키고, 널이 아닐 때에는 값을 적용시킨다.

```kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!!
    println(sNotNull.length)
}
```

* 널이 될 수 있는 타입에 항상 널이 아닌 값이 들어오는 경우 유용하게 사용할 수 있다.
* 널에 대해 !! 를 사용하여 예외가 발생할 때 어떤 식에서 발생했는지에 대한 정보가 없기 때문에 여러 !! 단언문을 한 줄에 쓰지 않는 것이 좋다.

```kotlin
person.company!!.address!!.country 
```

### let 함수

* let 함수는 자신의 수신 객체를 인자로 전달받은 람다에 넘긴다.
* 널이 아닌 타입을 인자로 받는 함수에 널이 될 수 있는 타입을 넘길 때 안전한 호출 구문과 함께 let 함수를 사용하면, 널이 아닌 타입으로 바꾸어 람다에 전달하게 된다.
* 아래와 같이 ?. 호출 연산자로 let 함수를 호출하면 email 값이 null이 아닐 때에만 람다가 수행된다.

```kotlin
email?.let { email -> sendEmailTo(email) }
email?.let { sendEmailTo(it) }
```

* 아래와 같이 함수의 결과를 변수에 저장할 필요 없이 바로 let 메서드를 통해 null이 아니면 람다를 수행하도록 할 수 있다.

```kotlin
getTheBestPersonInTheWorld()?.let { sendEmailTo(it.email) }
```

### 지연 초기화

* 널이 될 수 없는 프로퍼티에 대해서는 생성자가 아닌 별도 메서드에서 지연 초기화가 불가능하다.
* 따라서 지연 초기화 이후에 항상 널이 될 수 없는 필드를 널이 가능한 타입으로 두고 접근할 때 마다 널 검사를 하거나 `!!` 연산자를 붙여야 한다.
* &#x20;lateinit 변경자를 붙이면 프로퍼티를 나중에 초기화할 수 있다.
* 만약 lateinit 프로퍼티에 초기화 되기 전 접근하면 예외가 발생한다.
* 아래와 같이 JUnit을 사용할 때 myService 필드를 널이 될 수 없는 필드로 두고 lateinit 변경자를 붙인다. 그렇게 되면 최초에는 널로 할당되고, `@Before` 함수에서 해당 필드가 초기화된다.&#x20;

```kotlin
class MyTest {
    private lateinit var myService: MyService

    @Before fun setUp() {
        myService = MyService() 
    }

    @Test fun testAction() {
        Assert.assertEquals("foo", myService.performAction())
     }
}
```

### 널이 될 수 있는 타입 확장

* 널이 될 수 있는 타입에 대한 확장 함수를 정의하면 null 값을 다루는 강력한 도구로 활용될 수 있다.
* 코틀린에서는 String? 타입의 수신 객체에 대해 호출할 수 있는 isNullOrEmpty, isNullOrBlank 함수가 제공된다.

```kotlin
fun String?.isNullOrBlank(): Boolean =
    this == null || this.isBlank()
```

* 널을 검사하는 기능을 담은 확장 함수를 직접 작성할 수도 있다. 이 때 this는 널이 될 수 있으므로 명시적으로 널 여부를 검사해야 한다.

### 제네릭과 널 가능성

* 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다.
* 따라서 타입 파라미터를 사용할 때에는 널이 아닌지 확인한 후 사용해야 한다.

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashcode())
}
```

* 널이 될 수 없도록 타입 상한을 지정하면 널 검사를 하지 않아도 된다.

```kotlin
fun <T: Any> printHashCode(t: T) {
    println(t.hashcode())
}
```

### 자바와 널 가능성

* 자바에서는 JSR-305 표준이나 jetbrains 어노테이션 등에서 제공하는 `@Nullable`, `@NotNull`어노테이션으로 널이 가능한지 아닌지 표현할 수 있다.
* 코틀린에서는 이 어노테이션에 따라 널 가능 타입인지 확인한다.
* 코틀린이 널 관련 정보를 알 수 없는 타입은 플랫폼 타입이 된다. 플랫폼 타입은 널이 될 수 있는 타입으로 처리해도 되고 널이 될 수 없는 타입으로 처리해도 된다. 즉, 모든 연산의 책임을 사용자에게 두게 된다.

```kotlin
val s: String? = person.name
val s2: String = person.name
```

* 플랫폼 타입은 코틀린에서 생성할 수는 없으며, 널 가능성을 알지 못한다는 의미에서 `!` 가 붙는다. 예를 들어 자바에서 선언된 String 타입에 대해 코틀린이 널 가능성을 알 수 없다면 `String!` 타입으로 두게 된다.
* 코틀린에서 자바 클래스를 상속받아 메서드를 오버라이드할 때 파라미터와 반환타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언할 지 정할 수 있다.
* 아래는 java에 정의된 StringProcessor 인터페이스를 kotlin으로 상속받는 예제로, 메서드 파라미터를 널이 될 수 없는 String 타입으로 둘 수도 있고 널이 될 수 있는 String? 타입으로 둘 수도 있다.

```java
interface StringProcessor {
    void process(String value);
}
```

```kotlin
class StringPrinter: StringProcessor {
    override fun process(value: String) {
        println(value)
    }
}

class NullableStringPrinter: StringProcessor {
    override fun process(value: String?) {
        if (value != null) {
            println(value)
        }
    }
}
```

## 원시 타입

* 코틀린은 자바와 달리 원시 타입과 래퍼 타입을 구분하지 않는다.
* 코틀린의 Int 타입은 기본적으로 자바의 원시 타입인 int로 컴파일된다. 단, 컬렉션과 같은 제네릭 클래스를 사용할 경우 자바의 래퍼 타입인 Integer로 컴파일된다.
* 아래와 같이 널 값이나 널이 될 수 있는 타입을 사용하지 않더라도 Integer 타입 컬렉션으로 컴파일된다.

```kotlin
val listOfInts = listOf(1, 2, 3)
```

* 코틀린에서 널이 될 수 있는 타입 중 Int?, Boolean? 과 같은 타입은 int, boolean으로 컴파일될 수 없으므로 Integer, Boolean과 같은 래퍼 타입으로 컴파일된다.

### 숫자 변환

* 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다.
* 코드에서 동시에 여러 숫자 타입을 사용할 때 예상치 못한 동작을 피하기 위해 항상 변수를 명시적으로 변환해 사용해야 한다.

```kotlin
val x = 1
println(x.toLong() in listOf(1L, 2L, 3L)
```

### 원시 타입 리터럴

* 숫자 리터럴 종류는 아래와 같다.
  * Long 타입 리터럴: L 접미사를 붙인다. ex) 123L
  * Double 타입 리터럴: 표준 부동소수점 표기법을 사용한다. ex) 0.12, 1.2e-10
  * Float 타입 리터럴: f나 F 접미사를 붙인다. ex) 123.4f
  * 16진 리터럴: 0x나 0X 접두사를 붙인다. ex) 0x123CB, 0xbcdL
  * 2진 리터럴: 0b나 0B 접두사를 붙인다. ex) 0b0000101
* 숫자 리터럴 중간에 `_` 을 넣어 단위를 구분하기 쉽도록 할 수 있다. ex) 1\_000\_000
* 숫자 리터럴 사용 시 보통 변환 함수를 호출할 필요가 없다. 타입이 정해진 변수에 대입하거나 함수 인자로 넘기면 자동으로 변환 함수가 호출될 것이다.
* 산술 연산자의 경우 오버로딩이 되어 있어 여러 타입을 받아들일 수 있다.

```kotlin
val b: Byte = 1
val l = b + 1L
```

### Any, Any?

* 자바에서 Object가 모든 클래스의 최상위 타입이듯, 코틀린에서는 Any 타입은 널이될 수 없는 모든 타입의 최상위 타입이다.
* 자바의 원시 타입은 Object 타입이 아니지만 코틀린에서는 Int, Double 등 모든 타입이 Any 타입이다.
* Any 타입은 널이 될 수 없으므로 null이 들어갈 수 없다.
* 널을 포함하는 모든 값을 대입하려면 Any? 타입을 사용해야 한다.
* 내부적으로 Any 타입은 자바의 Object와 대응된다. Any 타입은 toString, equals, hashcode 메서드를 사용할 수 있지만 wait, notify 등 Object 클래스의 다른 메서드는 사용할 수 없다. 따라서 해당 메서드를 사용하고자 한다면 Object 타입으로 캐스팅해야 한다.

### Unit 타입

* 코틀린 함수를 자바의 void 메서드처럼 사용하려 할 때 Unit 타입을 반환하도록 하면 된다.
* Unit이란 단 하나의 인스턴스만 갖는 타입을 의미한다.
* Unit 타입은 모든 기능을 갖는 일반적인 타입이고, Unit이라는 값을 가진다.
* 아래 두 함수는 반환값이 없다는 의미를 동일하게 가진다.

```kotlin
fun f(): Unit {
    // ...
}

fun f() {
    // ...
}
```

* 제네릭 파라미터를 반환하는 함수를 오버라이드하면서 Unit을 반환할 수 있다.
* 아래는 제네릭 타입의 결과를 반환하지 않고 싶을 때 Unit 타입을 사용하는 예제이다. process 메서드는 결국 Unit 타입을 반환해야 하지만, 컴파일러가 자동으로 `return Unit` 를 넣어준다.

```kotlin
interface Processor<T> {
    fun process() : T
}
    
class NoResultProcessor : Processor<Unit> {
    override fun process() {
        // ...
    }
}
```

### Nothing 타입

* 아무 값도 포함하지 않으며 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로 사용되는 타입이다.
* 컴파일러는 Nothing이 반환 타입인 함수가 정상 종료되지 않는다는 것을 알고, 함수를 호출하는 코드를 분석할 때 사용한다.
* 아래와 같이 코틀린 테스트 시 제공되는 fail함수는 Nothing 타입을 반환한다. 컴파일러는 company.address가 널이라면 예외가 발생한다는 사실을 파악하고 address 값이 널이 아님을 추론한다.

```kotlin
fun fail(message: String) : Nothing {
    throw IllegalStateException(message)
}

val address = company.address ?: fail("No address")
```

## 컬렉션과 배열

### 널 가능성과 컬렉션

* 컬렉션 타입 인자 뒤에 `?`를 붙이면 컬렉션의 원소로 널을 저장할 수 있다. 이 때 리스트 자체는 널이 될 수 없다.

```kotlin
fun searchAll(list: List<Int?>): Boolean {
}
```

* 아래와 같이 사용하면 리스트 필드 자체에 널을 저장할 수 있다. 하지만 컬렉션 원소로 널을 저장할 수는 없다.

```kotlin
fun searchAll(list: List<Int>?): Boolean {
}
```

* 코틀린 표준 라이브러리에서는 filterNotNull이라는 함수를 제공해 null 원소들을 제외한 원소들만 구할 수 있다.

```kotlin
val validNumbers = numbers.filterNotNull() // List<Int> 타입이다.
```

### 읽기 전용 / 변경 가능한 컬렉션

* 코틀린 컬렉션에는 Collection 인터페이스와 MutableCollection 인터페이스가 존재한다.
* Collection 인터페이스에는 읽기 전용 연산들이 존재하며, MutableCollection 인터페이스는 Collection 인터페이스를 확장하고 원소 추가/제거 연산들을 가진다.

<figure><img src="../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

* 코드에서는 되도록 읽기 전용 인터페이스를 사용하고 필요할 때에만 변경 가능한 인터페이스를 사용해야 한다.
* 원본 컬렉션의 변경을 막기 위해 다른 함수에 전달할 때 복사본을 넘길 수도 있다.

```kotlin
fun<T> copyElements(source: Colleciton<T>, target: MutableCollection<T>) {
    for (item in source) {
        target.add(item)
    }
}

fun main(args: Array<String>) {
    val source: Collection<Int> = arrayListOf(3, 5, 7)
    val target: MutableCollection<Int> = arrayListOf(1)
    
    copyElements(source, target)
}
```

* 읽기 전용 컬렉션이라고 해서 변경 불가능한 컬렉션인 것은 아니다. 다른 참조에서 MutableCollection 타입으로 사용할 수도 있기 때문이다.

### 코틀린 컬렉션과 자바

* 다음은 코틀린 컬렉션 인터페이스의 계층 구조이다.
* 읽기 전용, 변경 가능 인터페이스의 기본 구조는 자바 컬렉션 인터페이스를 그대로 옮겨 놓은 형태이다.

<figure><img src="../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

* 코틀린에서는 자바의 ArrayList, HashSet 클래스를 MutableList, MutableSet을 상속받은 것처럼 취급한다.
* 컬렉션 생성 함수는 아래와 같이 나뉘게 된다.

| 컬렉션 타입 | 읽기 전용 타입 | 변경 가능 타입                                           |
| ------ | -------- | -------------------------------------------------- |
| List   | listOf   | mutableListOf, arrayListOf                         |
| Set    | setOf    | mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf  |
| Map    | mapOf    | mutableSetOf, hashMapOf, linkedSMapOf, sortedMapOf |

* 자바는 읽기 전용 컬렉션과 변경 가능 컬렉션을 구분하지 않으므로, 코틀린에서 읽기 전용 컬렉션을 자바 메서드 인자로 넘길 때 변경 가능해진다는 문제가 있다.
* 널이 아닌 원소로 이뤄진 컬렉션 타입을 자바 메서드 인자로 넘길 때에도 마찬가지로 컬렉션에 널이 들어갈 가능성이 있다.
* 이러한 문제를 해결하는 방법은 따로 없고, 사용자가 올바른 파라미터 타입을 넘겨 혼돈이 없도록 해야 한다.

### 컬렉션을 플랫폼 타입으로 다루기

* 코틀린은 자바에서 선언한 컬렉션 타입 변수를 플랫폼 타입으로 본다.
* 따라서 읽기 전용 컬렉션이나 변경 가능한 컬렉션 중 원하는 형태로 사용할 수 있다.
* 컬렉션 타입이 자바 메서드 시그니처에 있고 이를 코틀린에서 오버라이드하려 하면 마찬가지로 읽기 전용 컬렉션이나 변경 가능한 컬렉션 중 어떤 타입을 사용할 지 정해야 한다.
* 컬렉션이 널이 될 수 있는지, 원소로 널이 들어올 수 있는지, 오버라이드하는 메서드가 컬렉션을 변경하는지 등 자바에서 인터페이스나 클래스가 어떤 맥락에서 사용되는지 확인해 적절한 타입을 정해야 한다.

### 배열

* 타입 파라미터를 받으며 자바의 배열과 호환된다.
* arraysOf 함수에 여러 원소를 인자로 넣어 배열을 만들 수 있다.
* arraysOfNulls 함수에 정수 값을 인자로 넘겨 원하는 개수만큼 null이 담긴 배열을 만들 수 있다.
* 배열 크기와 람다를 인자로 받아 배열 원소를 초기화해주는 생성자도 존재한다.

```kotlin
val letters = Array<String>(26) { i -> ('a' + i).toString() }
```

* 코틀린에서 배열이 사용되는 상황은 다음과 같다.
  * 배열을 인자로 받는 자바 메서드 호출
  * vararg 인자를 받는 코틀린 함수 호출
* 컬렉션을 배열로 바꾸려면 toTypedArray 함수를 사용할 수 있다.
* 아래는 vararg 인자를 넘기기 위해 리스트 컬렉션을 배열로 변경한 후 스프레드 연산자 `*` 를 사용한 예제이다.

```kotlin
val strings = listOf("a", "b", "c")
println("%s/%s/%s".format(*strings.toTypedArray()))
```

* Array\<Int>는 제네릭 타입이기 때문에 자바의 Integer\[]로 변환된다.
* int\[]와 같은 원시 타입 배열로 변환되도록 하려면 IntArray, CharArray, BooleanArray와 같이 원시 타입 배열을 위한 별도 클래스를 사용해야 한다.
* 원시 타입 배열을 만드는 방법은 아래 세 가지가 있다.
  * 생성자에 size 인자를 입력해 고정 크기 배열을 만들 수 있다.
  * 팩토리 함수를 사용해 여러 원소를 인자로 넣어 배열을 만들 수 있다.
  * 크기와 람다를 인자로 받는 생성자로 배열을 만들 수 있다.

```kotlin
val arr1 = IntArray(5)
val arr2 = IntArrayOf(0, 0, 0, 0, 0)
val arr3 = IntArray(5) { i -> (i+1) * (i+1) }
```

* forEachIndexed 함수를 사용해 배열의 인덱스와 원소를 순회할 수 있다.

```kotlin
fun main(args: Array<String>) {
    args.forEachIndexed { index, element ->
        println("Argument $index is: $element")
    }
}
```
