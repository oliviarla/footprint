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

```kotlin
// 기본 형태
people.maxBy({ p: Persion -> p.age })
// 람다를 외부로 분리
people.maxBy() { p: Persion -> p.age }
// 빈 괄호 제거
people.maxBy { p: Persion -> p.age }
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





