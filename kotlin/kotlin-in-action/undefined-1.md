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



## 원시 타입

\
\
\
\
\
\


컬렉션과 배열\
\
\
\
\
\
\
\
\
\
\
\







