# item 68) 일반적으로 통용되는 명명 규칙을 따르라

##

## 명명 규칙

자바 플랫폼은 명명 규칙이 잘 정립되어 있으며, 대부분 자바 언어 명세에 기술되어 있다.

> 축약된 약자의 경우, 첫 글자만 대문자로 하는 경우가 많다.

| 식별자 타입     | 예                                                |
| ---------- | ------------------------------------------------ |
| 패키지와 모듈    | org.junit.jupiter.api, com.google.common.collect |
| 클래스와 인터페이스 | Stream, FutureTask, LinkedHashMap                |
| 메서드와 필드    | remove, groupingBy, getCrr                       |
| 상수 필드      | MIN\_VALUE, MAX\_VALUE                           |
| 지역변수       | i, denom, houseNum                               |
| 타입 매개변수    | T, E, K, V, X, R, U, V, T1, T2                   |

타입 매개변수 이름은 보통 한 문자로 표현한다.

1. 임의의 타입엔 `T`
2. 컬렉션 원소의 타입은 `E`
3. 맵의 키와 값은 `K`, `V`
4. 예외에는 `X`
5. 메서드 반환 타입에 `R`

## 메서드 이름 규칙

* 동사나 동사구로 짓는다.
* boolean 값 반환하는 경우 **is..., has...** 형태
* 반환 타입이 boolean이 아니거나 인스턴스의 속성 반환하는 경우 **명사, 명사구, get...** 형태
* 객체 타입을 바꾸는 (A to B) 경우 **to...** 형태 ex) toString, toArray, toDTO
* 객체의 내용을 다른 뷰에 보여주는 경우 **as...** 형태 ex) asList
* 객체의 값을 기본 타입 값으로 반환하는 경우 **...Value** 형태로 ex) intValue
* 정적 팩토리의 경우 **from, of, valueOf, instance, getInstace** 등의 형태
