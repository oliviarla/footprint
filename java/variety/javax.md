# Javax

{% hint style="info" %}
javax 패키지는 Java EE 기술 중 일부를 사용하기 위해 필요한 패키지 중 하나이다. Spring은 Java EE와 다른 프레임워크들과의 통합을 제공하기 위해 javax 패키지를 많이 활용하고 있다.
{% endhint %}

## Validation Constraints Annotation

`@NotNull` : 필드 값으로 Null을 허용하지 않음

`@NotEmpty` : 필드 값으로 Null, “” 허용하지 않음, String, Collection, Map, Array에 사용 가능

`@NotBlank` : 필드 값으로 null, “”, “ “ 허용하지 않음, String에만 사용 가능

숫자에만 사용 가능한 어노테이션

* `@Positive` : 필드 값이 0이 아닌 양수인지 확인
* `@PositiveOrZero` : 필드 값이 0 또는 양수인지 확인
* `@Negative` : 필드 값이 0이 아닌 음수인지 확인
* `@NegativeOrZero` : 필드 값이 0 또는 음수인지 확인
* `@Min(value=??)` : 필드 값이 value 이상인지 확인
* `@Max(value=??)` : 필드 값이 value 이하인지 확인

`@Email` : 필드 값이 유효한 이메일 주소인지 확인, null 허용

`@Size` : 필드 값이 min과 max 사이의 값인지 확인, String, Collection, Map, Array에 사용할 수 있다.

`@Valid` : Controller의 RequestBody에 적용해 Bean Validation 사용 가능

[https://www.baeldung.com/javax-validatio](https://www.baeldung.com/javax-validatio)
