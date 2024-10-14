# 람다

## 람다 식과 멤버 참조

### 람다 식

* 람다는 함수를 값처럼 다루어 코드를 함수에 넘기거나 변수에 저장하기 쉽게 해준다.
* 가장 간단한 람다 식은 아래와 같다. `->` 이전 부분은 파라미터 목록을 나타내고  `->` 이후 부분은 본문을 나타낸다. 항상 중괄호로 감싸진다.

```kotlin
{ x: Int, y: Int -> x + y }
```

* 람다 식은 변수에 저장하거나 직접 호출할 수 있다.

```kotlin
val sum = { x: Int, y: Int -> x + y }
val result = sum(1, 2)
```

```kotlin
val result = { x: Int, y: Int -> x + y }(1, 2)
```

* run에 람다를 인자로 넣어 실행시킬 수도 있다.

```kotlin
run { println("hello") }
```

* 함수 호출 시 맨 마지막 인자가 람다식이라면 람다를 괄호 밖으로 꺼낼 수 있다.
* 람다가 유일한 인자라면 빈 괄호를 없애도 된다.
* 일반적인 경우 파라미터 타입을 생략할 수 있다. 단, 람다식을 변수에 할당한다면 파라미터 타입을 생략하면 안된다.
* 람다의 파라미터가 하나뿐이고 타입을 컴파일러가 추론할 수 있다면 디폴트 파라미터 이름인 it를 사용할 수 있다. 단, 람다가 중첩되는 경우 등에는 파라미터가 어떤 람다에 속했는지 알기 어려우므로 직접 람다 파라미터를 명시해야 한다.

```kotlin
// 기본 형태
people.maxBy({ p: Person -> p.age })
// 람다를 외부로 분리
people.maxBy() { p: Person -> p.age }
// 빈 괄호 제거
people.maxBy { p: Person -> p.age }
// 파라미터 타입 생략
people.maxBy { p -> p.age }
// it 사용
people.maxBy { it.age }
```

* 아래는 함수 인자로 람다를 넣는 예시이다.

```kotlin
val people = listOf(Person("Alice", 21), Person("Bob", 32))

// 이름 붙인 인자를 사용하기
val result = people.joinToString(separator = " ", tramsform = { p: Person -> p.name })

// 람다를 함수 인자 외부로 분리하기
val result = people.joinToString(" ") { p: Person -> p.name }
```

* 본문이 여러 문장으로 구성된 람다식의 경우 맨 마지막 식이 람다의 결과값이 된다.

```kotlin
val sum = { x: Int, y: Int ->
    println("computing...")
    x + y
}

val result = println(sum(1, 2))
```

### 컬렉션에서 활용

* 사람 정보를 담은 리스트에서 나이를 기준으로 가장 많은 사람을 구하려면 for문을 순회할 수 있다. 람다와 코틀린에서 제공되는 maxBy 함수를 사용하면 동일한 내용을 간단히 구현할 수 있다.
  * `it` 는 컬렉션의 원소를 인자로 받았을 때를 의미한다.

```kotlin
// for문 사용
val people = listOf(Person("Alice", 21), Person("Bob", 32))
var maxAge = 0
var theOldest: Person? = null
for (person in people) {
    if (person.age > maxAge) {
        maxAge = person.age
        theOldest = person
    }
}
println(theOldest)

// 람다와 maxBy 함수 사용
println(people.maxBy({ p: Persion -> p.age }))
println(people.maxBy { it.age })

// 멤버 참조와 maxBy 함수 사용
println(people.maxBy(Person::age))
```

### 현재 영역의 변수 접근

* 자바의 익명 클래스에서는 익명 클래스를 정의한 메서드의 로컬 변수를 사용할 수 있듯이, 람다에서도 람다를 정의한 함수의 파라미터, 로컬 변수를 사용할 수 있다.

```kotlin
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
}
```

* 자바와 달리 파이널 변수가 아닌 로컬 변수여도 접근이 가능하다.
* 함수 내에 정의된 로컬 변수의 생명 주기는 함수가 반환되면 끝나지만, 람다에서 포획하였을 경우 함수가 끝난 뒤에도 읽거나 쓸 수 있기 때문에 생명 주기가 달라질 수 있다.
* 람다가 변경 가능한 변수(var)를 포획할 경우 Ref 객체에 담아 해당 참조를 final로 만들어 사용하도록 한다. Ref 객체 내부의 값은 바뀌어도 되기 때문에 코틀린에서 제공하는 람다에서는 메서드 로컬 변수를 바꾸는 것 처럼 동작한다.
* 만약 람다가 비동기적으로 동작하여 함수 호출이 끝난 후에 로컬 변수가 변경된다면 클래스/전역 프로퍼티로 두어 변수의 변화를 추적할 수 있도록 해야한다.
* 아래와 같이 작성할 경우 버튼을 클릭했을 때 람다식이 실행된다. 하지만 함수가 반환된 후에 버튼을 클릭하면 클릭수를 반영할 수 없다.

```kotlin
fun tryToCountButtonClicks(button: Button) : Int {
    var clicks = 0
    button.onClick { clicks++ }
    return clicks
}
```

* 만약 람다 자신을 가리키는 것이 필요하다면 이는 불가능하기 때문에 익명 객체를 사용해야 한다.

### 멤버 참조

* 클래스에 이미 정의된 함수를 전달하려면 멤버 참조를 이용할 수 있다.
* 아래와 같이 클래스 이름과 참조하려는 프로퍼티 혹은 메서드 이름 사이에 `::` 를 입력하면 멤버 참조가 된다.

```kotlin
Person::age
```

* 확장 함수도 멤버 함수와 같은 방식으로 참조 가능하다.

```kotlin
fun Person.isAdult() = age >= 21
val predicate = Person::isAdult
```

* 최상위에 선언된 함수나 프로퍼티는 아래와 같이 참조할 수 있다.

```kotlin
fun greetings() = println("안녕하세요.")

fun main(args:Array<String>) {
    run(::greetings)
}
```

* 함수의 인자가 많을 경우 람다를 전달하는 것보다 위임 함수에 대한 참조를 제공하는 것이 하나하나 인자를 전달할 필요 없으므로 간편하다.
* 생성자 참조를 이용해 클래스 생성 작업을 저장할 수 있다. `::클래스이름` 형태로 선언할 수 있다.

```kotlin
val createPerson = ::Person
val person = createPerson("Alice", 21)
```

## 컬렉션의 함수형 API

#### filter

* 컬렉션에서 조건을 만족하는 원소만을 남기기 위한 함수이다.
* 아래는 사람의 정보를 담은 리스트에서 나이가 30을 초과하는 경우만 리스트로 반환하는 예제이다.

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.filter { it.age > 30 })
```

#### map

* 컬렉션의 모든 원소를 특정 형태로 변형시키기 위한 함수이다.
* 아래는 사람의 정보를 담은 리스트를 사람의 이름만을 담는 리스트로 반환하는 예제이다.

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
people.map( {it.name} )
```

* Map 타입일 경우 key-value 형태이기 때문에 각각 적용하기 위한 filterKeys, mapKeys, filterValues, mapValues 함수가 존재한다.

#### all, any

* 컬렉션의 모든 원소가 조건을 만족하거나 일부 원소가 조건을 만족하는지 확인하기 위한 함수이다.

```kotlin
val predicate = { p: Person -> p.age <=20 }
val people = listOf(Person("Alice", 19), Person("Bob", 31))

println(people.all(predicate)) // false
println(people.any(predicate)) // true
```

#### count

* 조건을 만족하는 원소의 개수를 반환하는 함수이다.
* filter 함수를 통해 만들어진 컬렉션의 크기를 구하는 것보다, count 함수를 사용하는 것이 효율적이다.

```kotlin
val predicate = { p: Person -> p.age <=20 }
val people = listOf(Person("Alice", 19), Person("Bob", 31))

println(people.count(predicate)) // 1
```

#### find

* 조건을 만족하는 첫 번째 원소를 반환하는 함수이다. 조건을 만족하는 원소가 없으면 null을 반환한다.

```kotlin
val predicate = { p: Person -> p.age <=20 }
val people = listOf(Person("Alice", 19), Person("Bob", 31))

println(people.find(predicate)) // Person(name=Alice, age=19)
```

#### groupBy

* 원소의 특정 속성에 따라 구분하여 Map 형태로 반환하는 함수이다.
* 아래는 Person 리스트를 나이 기준으로 분류하여 Map\<Int, List\<Person>> 타입으로 반환하는 예제이다.

```kotlin
val people = listOf(Person("Alice", 19), Person("Bob", 31), Person("Noah", 19))
println(people.groupBy { it.Age })
```

#### flatMap, flatten

* flatMap 함수는 인자로 주어진 람다를 컬렉션의 모든 원소에 적용한 결과 생성되는 리스트들을 하나로 모은다.
* 아래는 toList 함수를 사용해 문자열에 속한 문자를 리스트로 만들어버린 후, flatMap으로 리스트들을 하나로 합치는 예제이다.

```kotlin
val strings = listOf("abc", "d", "ef")
val newMap = strings.flatMap { it.toList() } // [a, b, c, d, e, f]
```

* 리스트 안에 리스트들이 존재하는 중첩 리스트를 하나의 리스트로 모아야 하고 변환 과정은 필요 없을 땐 flatten 함수를 사용하면 된다.

```kotlin
val listOfLists = listOf(listOf("candy", "jelly"), listOf("noodle", "rice"))
val list = listOfLists.flatten()
```

## 지연 계산 연산

* 앞서 살펴본 컬렉션의 함수형 API는 즉시 결과 컬렉션을 생성해 반환한다. 따라서 컬렉션 함수를 체이닝하면 매 단계마다 중간 컬렉션을 만들게 된다.
* **시퀀스**를 이용하면 자바의 stream과 유사하게 지연 계산 방식으로 동작한다. 즉, map, filter 함수를 체이닝하더라도  원본 컬렉션에 일괄적으로 map, filter 함수를 적용하는 것이 아니라 원소 하나씩 함수를 적용한다.
* 지연 계산 연산을 통해 즉시 연산에 비해 성능을 향상시킬 수 있다. 예를 들어 map 함수와 find 함수를 체이닝했다면 즉시 연산 시에는 모든 원소에 map 함수를 적용하여 컬렉션을 만들고 해당 컬렉션을 find 함수에 입력할 것이다. 반면 지연 연산 시에는 첫 번째 원소를 map함수, find 함수에 적용해보고 조건이 맞다면 다음 원소는 접근하지 않는다.
* 시퀀스에 대한 연산은 다른 시퀀스를 반환하는 **중간 연산**과 결과를 반환하는 **최종 연산**으로 나뉜다.
* `asSequence` 메서드를 통해 컬렉션 등을 시퀀스로 변환할 수 있다.
* 시퀀스를 다시 컬렉션으로 만들려면 toXXX 함수를 사용해야 한다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map { print("map($it) "); it * it }
    .filter { print("filter($it) "); it % 2 == 0 }
    .toList()
```

* `generateSequence` 메서드를 통해 시퀀스 객체를 만들 수 있다.

```kotlin
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
val result = numbersTo100.sum() // 5050
```

## 자바의 함수형 인터페이스 활용

* 추상 메서드가 하나만 있는 인터페이스를 함수형 인터페이스 혹은 SAM(Single Abstract Method) 인터페이스라고 부른다.
* 코틀린에서 인자가 함수형 인터페이스인 자바 메서드를 호출 시 람다를 넘길 수 있게 해준다.
* 익명 객체를 넘기게 되면 함수를 호출할 때 마다 새로운 객체가 생성되지만, 람다를 넘기면 반복해서 재사용할 수 있다. 단, 람다가 주변 영역의 변수를 포획한다면 컴파일러는 매번 새로운 객체를 생성해준다.
* 아래는 익명 객체를 넘기는 것과 람다를 넘기는 것의 예제이다.

```kotlin
postponComputation(1000, object : Runnable {
    override fun run() {
        println(42)
    }
})
```

```kotlin
postponComputation(1000) { println(42) }
```

### SAM 생성자

* SAM 생성자는 람다를 함수형 인터페이스 객체로 변환할 수 있게 컴파일러가 자동 생성한 함수이다.
* 함수형 인터페이스 타입을 반환하는 메서드의 경우 람다를 직접 반환하는 것은 불가능하며, 람다를 SAM 생성자로 감싸야 한다.
* 혹은 람다로 생성한 함수형 인터페이스 객체를 변수에 저장하려는 경우도 SAM 생성자를 사용할 수 있다.
* SAM 생성자의 이름은 함수형 인터페이스의 타입과 같다.

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") } // SAM 생성자 사용해 Runnable 타입 객체 생성
}
```

* 오버로드한 메서드 중 어떤 타입의 메서드로 람다를 변환해 넘겨줘야 할 지 모호한 경우 명시적으로 SAM 생성자를 적용해 컴파일 오류를 피할 수 있다.

## 수신 객체 지정 람다

* 수신 객체를 명시하지 않고 람다 본문 안에서 다른 객체의 메서드를 호출할 수 있도록 하는 것을 의미한다.

### with

* 하나의 객체를 여러번 사용하는 경우 계속해서 변수명을 반복해 작성해주어야 한다.
* with 함수를 이용하면 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만들어 람다 본문에서 this로 객체에 접근할 수 있도록 해준다.
* with 함수는 람다 코드를 실행한 결과를 반환한다. 따라서 람다의 가장 마지막 식의 값이 반환될 것이다.
* 아래는 stringBuilder를 첫 번째 인자로 받고, 람다를 두 번째 인자로 받아 문자열을 만들어내는 예제이다.

```kotlin
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for (letter in 'A'..'Z') {
            this.append(letter)
        }
        append("\\nNow I know the alphabet!")
        toString()
    }
}
```

* with 함수를 사용하는 클래스와 with에게 인자로 넘긴 객체의 클래스에 동일한 이름을 가진 메서드들이 존재한다면 this 참조 뒤에 레이블을 붙여야 한다.

```kotlin
this@OuterClass.toString()
```

### apply

* with와 거의 유사하지만 자신에게 전달된 수신 객체를 그대로 반환한다.
* apply는 확장 함수로 정의되어 있다.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\\nNow I know the alphabet!")
}.toString()
```

* 객체를 생성하는 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 apply 함수에 넘겨 프로퍼티를 설정하거나 메서드를 호출할 수 있다.

```kotlin
fun createViewWithCustomAttributes(context: Context) =
    TextView(context).apply {
        text = "sample"
        textSize = 20.0
        setPadding(10, 0, 0, 0)
    }
```

* buildString의 경우 수신 객체가 항상 StringBuilder가 되며 수신 객체 지정 람다를 입력받는다.

```kotlin
fun alphabet() = buildString {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\\nNow I know the alphabet!")
}
```
