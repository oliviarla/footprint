# item 61) 박싱된 기본타입보단 기본 타입을 사용하라

## 기본 타입과 박싱된 기본 타입의 차이점

* 기본 타입은 값만 가지고 있지만, 박싱된 기본 타입은 값에 더해 식별성이라는 속성을 가진다.
* 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 **유효하지 않은 값(null)을 가질 수 있다.**
* 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용 면에서 효율적이다.
* 박싱된 기본 타입에 == 비교 연산자를 사용 시 주의해야 한다. 두 객체 참조의 식별성을 검사하기 때문에, 내부 값이 같아도 다른 인스턴스라면 다르다고 결과가 나온다.
* 박싱된 기본 타입의 초기값은 다른 참조 타입 필드와 마찬가지로 null이다. 기본 타입의 초기값과는 다를 것이다.
* 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는, 박싱이 자동으로 풀려 연산이 수행된다.

## 박싱된 기본 타입 사용하는 경우

* 기본 타입을 지원하지 않는 "**컬렉션의 원소, 키, 값, 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수**"에서 사용해야 한다.
* reflection(item 65)을 통해 메서드 호출 시에 박싱된 기본 타입을 사용해야 한다.