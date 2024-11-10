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













