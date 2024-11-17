# 어노테이션과 리플렉션

## 어노테이션 선언과 적용

### 어노테이션 적용

* 어노테이션을 적용하는 것은 직접 만드는 것보다 훨씬 쉽다.
* 적용하려는 대상 앞에 어노테이션을 붙이기만 하면 된다.
* 어노테이션에 인자를 넘길 때에는 일반 함수와 마찬가지로 괄호 안에 인자를 넣는다.
* 원시 타입 값, 문자열, enum, 클래스 참조, 다른 어노테이션 클래스, 배열이 들어갈 수 있다.

```kotlin
@Deprecated("use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
fun remove(index: Int) { ... }
```

* 어노테이션 인자를 지정하는 문법에서 특이한 점은 다음과 같다.
  * 클래스를 인자로 지정할 때에는 `::class`를 클래스 이름 뒤에 붙여야 한다.
  * 다른 어노테이션을 인자로 지정할 때에는 앞에 `@`를 붙이지 않는다.
  * 배열을 인자로 지정할 때에는 arrayOf 함수를 사용한다.&#x20;
    * ex) `@KotlinAnnotation(arrayOf("foo", "var"))`
    * 자바에서 선언한 어노테이션 클래스를 사용한다면 가변 길이 인자로 변환되므로 arrayOf 대신 `@JavaAnnotation("foo", "var")`와 같이 인자들을 어노테이션에 입력하면 된다.
* 어노테이션 인자는 컴파일 시점에 알 수 있어야 하므로 const 프로퍼티가 아닌 일반 프로퍼티는 인자로 지정할 수 없다.
* 자바 어노테이션에서는 어노테이션의 인자로 value를 제외한 모든 애트리뷰트에는 이름을 명시해야 한다. 코틀린 어노테이션에서는 인자의 이름을 명시할 수도 있고 생략할 수도 있다.

### 어노테이션 대상

* **사용 지점 대상 선언**을 통해 어노테이션을 붙일 요소를 정할 수 있다.
* 예를 들어 `@get:Rule` 은 Rule 어노테이션을 프로퍼티 게터에 적용하라는 의미이다.
  * Rule 어노테이션은 필드에 적용되지만, 코틀린의 필드는 기본적으로 비공개이므로 `@folder:Rule` 과 같이 정의하면 공개 필드가 아니라는 예외가 발생한다.
  * Rule 어노테이션을 TemporaryFolder와 함께 사용하면 메서드가 끝나면 삭제할  임시 파일과 폴더를 사용하도록 해준다.

```kotlin
class HasTempFolder {
    @get:Rule
    val folder = TemporaryFolder()
    
    @Test
    fun testUsingTempFolder() {
        val createdFile = folder.newFile("myfile.txt")
        // ...
    }
}
```

* 사용 지점 대상을 지정할 때 지원하는 목록은 다음과 같다.
  * `property` (annotations with this target are not visible to Java)
  * `field`
  * `get` (property getter)
  * `set` (property setter)
  * `receiver` (receiver parameter of an extension function or property)
  * `param` (constructor parameter)
  * `setparam` (property setter parameter)
  * `delegate` (the field storing the delegate instance for a delegated property)
  * `file` (package 선언 앞에서 파일의 최상위 수준에만 적용)
    * 주로 파일의 최상위 선언을 담는 클래스 이름을 바꿔주는 @JvmName과 함께 쓰인다. ex) `@file:JvmName("StringFunctions")`
* 자바와 달리 코틀린의 어노테이션 인자로는 클래스나 함수 선언이나 타입 외에 임의의 식도 입력할 수 있다.
* 코틀린으로 선언한 내용을 자바 언어의 특정 키워드를 붙여 컴파일하기 위해 아래와 같은 어노테이션들이 제공된다.
  * `@JvmName`: 자바 필드나 메서드 이름을 변경한다.
  * `@JvmStatic`: 정적으로 변경한다.
  * `@JvmOverloads`: 디폴트 파라미터 값이 있는 함수에 대해 오버로딩한 함수를 생성해준다.
  * `@JvmField`: getter/setter가 없는 public 자바 필드를 만들어준다.

### 메타 어노테이션

* 어노테이션에 적용할 수 있는 어노테이션 클래스이다.
* 표준 라이브러리 메타 어노테이션 중 가장 흔히 쓰이는 것은 @Target 이다. 어노테이션을 적용할 수 있는 요소의 유형을 지정한다. 프로퍼티, 클래스, 메서드 등의 유형을 지정할 수 있으며 여러 대상을 한꺼번에 지정할 수도 있다.
* 직접 메타 어노테이션을 만드려면 아래와 같이 ANNOTATION\_CLASS 를 붙여주면 된다.

```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)
annotation class BindingAnnotation
```

## 리플렉션

* 리플렉션은 실행 시점에 객체의 프로퍼티와 메서드에 접근할 수 있게 해준다.
* 보통 컴파일 시점에 구체적인 선언을 통해 객체의 메서드나 프로퍼티를 가리키면 이를 컴파일러가 실제 존재함을 보장하게 된다.
* 타입과 관계 없이 객체를 다뤄야 하거나 객체가 제공하는 메서드나 프로퍼티 이름을 런타임에만 알 수 있는 경우 리플렉션을 사용해야 한다.
* 코틀린 클래스는 자바 바이트코드로 컴파일되므로 java.lang.reflect 패키지를 통해 리플랙션을 사용할 수 있다.
* kotlin.reflect 패키지를 통해 자바에 없는 프로퍼티, 널이 될 수 있는 타입 등 코틀린만의 개념에 대한 리플렉션을 제공한다.

### 코틀린 리플렉션 API

*   리플렉션을 위해 사용 가능한 인터페이스의 계층 구조는 다음과 같다.

    * KClass는 클래스와 객체를 표현한다.
    * KProperty는 프로퍼티를 표현한다.
    * KMutableProperty는 var로 정의한 변경 가능한 프로퍼티를 표현한다.
    * KFunction은getter, setter를 비롯한 다양한 함수들을 표현한다.

    <figure><img src="../../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

#### KClass

* 클래스 내부의 모든 선언을 순회할 수 있으며, 각 선언에 접근하거나 클래스의 상위 클래스에 접근할 수도 있다.
* 런타임에 객체의 클래스를 얻으려면, 객체의 JavaClass 프로퍼티를 이용해 자바 클래스를 얻은 후 .kotlin 확장 프로퍼티를 통해 KClass를 얻을 수 있다.
* simpleName 프로퍼티를 통해 클래스 이름을 가져올 수 있다.
* memberProperties 프로퍼티를 통해 클래스와 모든 부모 클래스에 정의된 프로퍼티를 가져올 수 있다.

```kotlin
import kotlin.reflect.full.*

val person = Person("Alice", 20)
val kClass = person.javaClass.kotlin
val name = kClass.simpleName // Person
val properties = kClass.memberProperties.forEach { it.name } // [ "age", "name" ]
```

* KClass 인터페이스 선언은 다음과 같다.

```kotlin
interface KClass<T : Any> {
    val simpleName: String?
    val qualifiedName: String?
    val members: Collection<KCallable<*>>
    val constructors: Collection<KFunction<T>>
    val nestedClasses: Collection<KClass<*>>
    ...
}
```

#### KCallable

* 함수와 프로퍼티를 아우르는 공통 상위 인터페이스이다.
* call 함수를 사용하여 함수나 프로퍼티의 게터를 호출할 수 있다.
* call 함수를 호출 시 원래 함수에 정의된 파라미터 개수와 인자 개수가 맞아야 한다.
* call 함수는 모든 타입의 함수에 적용할 수 있지만 타입 안전성을 보장해주지 않는다.

```kotlin
fun foo(x:Int) = println(x)
val kCallable = ::foo
kCallable.call(42)
```

```kotlin
interface KCallable<out R> {
    fun call(vararg args: Any?): R
    fun callBy(args: Map<KParameter, Any?>): R
}
```

* callBy 메서드를 사용하여 디폴트 생성자 파라미터 값이 있다면 해당 값을 사용해 생성할 수 있다.

#### KFunction

* 구체적인 파라미터와 반환 값 타입 정보가 들어간다.
* invoke 메서드를 호출하면 KFunctionN 인터페이스를 통해 함수를 호출할 수 있다.
* invoke 메서드를 호출하지 않고 KFunctionN 객체에 인자를 넣어 함수처럼 직접 호출할 수 있다.

```kotlin
fun sum(x: Int, y: Int) = x + y
val kFunction: KFunction2<Int, Int, Int> = ::sum
val result = kFunction.invoke(1, 2) + kFunction(3, 4)
```

#### KProperty

* call 메서드를 통해 프로퍼티의 게터를 호출할 수 있다.
* get 메서드 역시 제공되어, 프로퍼티 값을 가져올 수 있다.
* 멤버 프로퍼티는 KProperty1 객체이며, 해당  객체에는 인자가 1개인 get 메서드가 들어있다.
* KProperty1은 제네릭 클래스로, 첫번째 타입 파라미터는 수신 객체 타입, 두 번째 타입 파라미터는 프로퍼티 타입을 표현한다.

```kotlin
val person = Person("Alice", 20)
val memberProperty: KProperty<Person, Int> = Person::age
val age = memberProperty.get(person)
```

* 함수의 로컬 변수에는 리플렉션으로 접근할 수 없다.
* 다음과 같이 여러 프로퍼티  중 특정 어노테이션이 붙은 프로퍼티를 제외할 수도 있다.

```kotlin
val properties = kClass.memberProperties
    .filter { it.findAnnotation<JsonExclude>() == null }
```
