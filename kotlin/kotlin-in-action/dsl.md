# DSL 만들기

## DSL

### DSL의 필요성

* API를 깔끔하게 작성하는 것은 모든 개발자에게 중요한 일이다. 이름을 적절히 붙이고 적절한 개념을 사용해 코드를 작성해야 한다. 그리고, 불필요한 구문이나 번잡한 준비 코드 없이 코드가 간결해야 한다.
* 코틀린은 확장 함수, 중위 호출, 연산자 오버로딩, get 메서드에 대한 관례, 람다를 괄호 밖으로 빼내는 관례, 수신 객체 지정 람다 등을 지원하여 깔끔하게 API를 작성할 수 있도록 한다.

```kotlin
StringUtil.capitalize(s) -> s.capitalize()

l.to("one") -> l to "one"

set.add(2) -> set += 2

map.get("key") -> map["key"]

file.use({f -> f.read()}) -> files.use {it.read()}

sb.append("yes") sb.append("no") -> with (sb) { append("yes") append("no") }
```

* 코틀린은 한 걸음 더 나아가 DSL 구축을 도와주는 다양한 기능을 제공한다.&#x20;
* DSL은 메서드 호출만을 제공하는 API에 비해 더 직관적인 인터페이스를 제공한다.

### DSL이란

* Domain Specific Language의 약자로, SQL, 정규식과 같이 특정 작업에 적합하도록 작성된 언어이다.
* 특정 영역에 필요하지 않은 기능들은 모두 없애 단순하고 간결하게 로직을 작성할 수 있다.
* DSL은 범용 언어로 만든 애플리케이션과 통합되기 어렵다. DSL로 작성된 프로그램을 다른 언어에서 호출하려면 별도 파일이나 문자열 리터럴로 저장해야 하는데, 이로 인해 컴파일 시점에 검증하거나 프로그램을 디버깅하기 어려워진다.
* 이러한 단점을 보완하기 위해 범용 언어로 작성할 수 있는 내부 DSL 개념이 생겨났다.
* 코틀린은 자바의  QueryDSL과 같이 exposed라는 SQL 프레임워크를 제공하여 코틀린 함수를 이용해 SQL 질의를 작성할 수 있도록 하는데, 이는 내부 DSL 중 하나이다.
* 코틀린의 내부 DSL은 보통 람다를 중첩시키거나 메서드 호출을 연쇄시키는 구조를 사용한다.

## 수신 객체 지정 DSL

* 수신 객체를 지정하면 메서드 이름 앞에 객체를 나타내는 `it` 를 명시하지 않고 `this` 처럼 사용할 수 있다.
* 아래는 수신 객체를 지정하지 않아 직접 it를 명시해주어야 하는 예제이다.

```kotlin
// 수신 객체 지정하지 않은 함수
fun buildString(
    builderAction: (StringBuilder) -> Unit
): String {
    val sb = StringBuilder()
    builderAction(sb) // stringBuilder 객체를 람다에 넘긴다.
    return sb.toString()
}

val s = buildString {
    it.append("a")
    it.append("b")
}
```

* 아래는 확장 함수 타입을 사용해 `.` 앞에 오는 타입을 수신 객체 타입으로 지정하고, 괄호 안에는 파라미터 타입을, `->` 뒤에는 반환 타입을 지정하여 수신 객체 지정 람다를 사용한 예제이다.

```kotlin
// 수신 객체 지정한 함수
fun buildString(
    builderAction: StringBuilder.() -> Unit // 확장 함수 타입을 통해 수신 객체 지정 람다 정의
) : String {
    val sb = StringBuilder()
    sb.builderAction() // StringBuilder 객체를 람다의 수신 객체로 넘긴다.
    return sb.toString()
}

val s = buildString {
    append("a")
    append("b")
}
```

* 수신 객체 지정 람다를 이용하면 **코드가 간결해지며, 특정 함수 내부에서만 함수를 사용할 수 있도록 강제할 수 있다.**
* 예를 들어 코틀린의 HTML 빌더는 다음과 같이 간단한 표를 만들 수 있는데, table 함수에 넘기는 람다 내부에서만 tr 함수를 사용할 수 있다. td 함수도 마찬가지로 tr 안에서만 접근 가능한 함수이다.

```kotlin
fun createSimpleTable() = createHTML().
    table {
        tr {
            td { +"cell" }
        }
    }
```

* 다음은 HTML 빌더에서 사용되는 유틸리티 클래스로, 각각자신의 내부에 들어갈 수 있는 태그를 생성하는 메서드를 가진다.

```kotlin
fun table(init: TABLE.() -> Unit) = TABLE().apply(init)

class TABLE : Tag("table") {
    fun tr(init: TR.() -> Unit) = doInit(TR(), init) // TR 타입을 수신 객체로 받는 람다를 인자로 받음
}

class TR : Tag("tr") {
    fun td(init: TD.() -> Unit) = doInit(TD(), init) // TD 타입을 수신 객체로 받는 람다를 인자로 받음
}

class TD : Tag("td")
```

* 수신 객체를 직접 명시하는 경우 아래와 같이 코드가 복잡해진다.  일반 람다를 사용한다면 더욱 복잡해질 것이다.

```kotlin
fun createSimpleTable() = createHTML().
    table {
        (this@table).tr {
            (this@tr).td { +"cell" }
        }
    }
```

* 수신 객체 지정 람다를 중첩할 경우 내부 람다에서 외부 람다에 정의된 수신 객체를 사용할 수 있다. 위 코드 예제에서 tr 함수 인자로 입력된 람다에서 tr 객체를 사용하는 것을 확인할 수 있다.
* 멤버 확장 개념과 함께 수신 객체 지정 람다를 사용해 자신의 수신 객체를 다시 반환하여 **메서드를 연쇄 호출**할 수 있다.
* 멤버 확장으로 선언하면 해당 클래스 외부에서는 메서드를 적용할 수 없도록 하며,  수신 객체의 타입을 제한할 수 있다.

```kotlin
class Table {
    fun integer(name: String): Column<Int>
    fun varchar(name: String, length: Int): Column<String>
    // ...
    
    // 멤버 확장
    fun <T> Column<T>.primaryKey(): Column<T>
    fun <T> Column<Int>.autoIncrement(): Column<T>
}
```

```kotlin
object Country: Table() {
    val id = integer("id").autoincrement().primaryKey()
    val name = varchar("name", 50)
}
```

## invoke 관례를 사용한 블록 중첩

* invoke 관례를 사용하면 객체를 함수처럼 호출할 수 있다. operator 변경자가 붙은 invoke 메서드 정의를 클래스에 추가하면 된다.

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}
```

* 인라인하는 람다를 제외한 모든 람다는 함수형 인터페이스를 구현하는 클래스로 컴파일된다. 각 함수형 인터페이스 안에는 invoke 메서드가 포함되어 있다.
* 이러한 원리 덕분에 복잡한 람다를 여러 메서드로 분리하면서, 분리 전 람다처럼 외부에서 호출 가능한 객체로 만들 수 있다.
* 다음은 특정 이슈의 중요도를 필터링하는 람다를 클래스로 두고 중요도 판별 로직을 내부 메서드로 분리한 후 invoke 관례를 통해 람다처럼 사용할 수 있도록 하는 예제이다.

<pre class="language-kotlin"><code class="lang-kotlin">data class Issue(
    val id: String, val project: String, val type: String,
    val priority: String, val description: String
)

class ImportantIssuesPredicate(val project: String)
        : (Issue) -> Boolean {

    override fun invoke(issue: Issue): Boolean {
        return issue.project == project &#x26;&#x26; issue.isImportant()
    }

    private fun Issue.isImportant(): Boolean {
        return type == "Bug" &#x26;&#x26;
                (priority == "Major" || priority == "Critical")
    }
}

val i1 = Issue("IDEA-154446", "IDEA", "Bug", "Major",
                   "Save settings failed")
val i2 = Issue("KT-12183", "Kotlin", "Feature", "Normal",
"Intention: convert several calls on the same receiver to with/apply")
<strong>val predicate = ImportantIssuesPredicate("IDEA")
</strong>val filtered = listOf(i1, i2).filter(predicate)
</code></pre>

* invoke 관례를 이용하면 중첩된 블록 구조를 구현할 수 있다.
* 다음은 gradle DSL 예제로, depencies라는 객체의 invoke 관례를 이용해 수신 객체 지정 람다를 인자로 넘겨 중첩된 블록 구조를 사용하는 형태이다.

```kotlin
val dependencies = DependencyHandler()

dependencies.compile("junit:junit:4.11")

// 같은 내용을 중첩된 블록 구조로 선언
dependencies {
    compile("junit:junit:4.11")
}
```

```kotlin
class DependencyHandler {
    fun compile(coordinate: String) {
        println("Added dependency on $coordinate")
    }

    operator fun invoke(
            body: DependencyHandler.() -> Unit) {
        body()
    }
}
```

## 중위 호출 연쇄

* 코틀린 테스트 DSL에서는 중위 호출 연쇄를 잘 지원하고 있다.
* 아래와 같이 should 함수와 Matcher를 구현한 객체를 이용하여 영어 문장처럼 테스트 검증 코드를 작성할 수 있다.

```kotlin
infix fun <T> T.should(matcher: Matcher<T>) = matcher.test(this)

interface Matcher<T> {
    fun test(value: T)
}

class startWith(val prefix: String): Matcher<String> {
    override fun test(value: String) {
        if (!value.startsWith(prefix))
            throw AssertionError("String $value does not start with $prefix")
    }
}
```

```kotlin
val s = "kotlin"
s should startWith("kot")
```

* 아래와 같이 조금 더 복잡한 구현을 통해 중위 호출을 연쇄시킬 수도 있다. object로 선언한 타입을 파라미터 타입으로 사용하여 should를 오버로딩한 함수들 중 start에 정의된 함수를 사용할 수 있다.

```kotlin
object start

infix fun String.should(x: start): StartWrapper = StartWrapper(this)

class StartWrapper(val value: String) {
    infix fun with(prefix: String) =
        if (!value.startsWith(prefix))
            throw AssertionError("String $value does not start with $prefix")
        else
            Unit
}
```

```kotlin
"kotlin" should start with "kot"
```

* 단, DSL은 여전히정적 타입 지정 언어이기 때문에 함수와 객체를 잘못 조합하면 컴파일 에러가 발생한다.
