# item 56) 공개된 API 요소에는 항상 문서화 주석을 작성하라

## 자바독(javadoc)

* API 문서화 유틸리티
* 소스코드 파일에서 문서화 주석이라는 형태로 기술된 설명을 추려 API문서로 변환
* 자바 버전에 따라 자바독 태그가 새롭게 추가된다.
  * 자바5 - @literal @code
  * 자바8 - @implSpec
  * 자바9 - @index

## API 문서를 올바르게 쓰는 법

* 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술 해야함
* 메서드가 '어떻게' 동작하는지가 아닌, ‘무엇’을 해야하는지 기술
* 메서드를 호출하기 위한 **전제조건**을 모두 나열
* @throws 태그로 비검사 예외를 선언해 암시적으로 기술
* 메서드가 성공적으로 수행된 후 만족해야 하는 **사후조건**을 모두 나열
* @param 태그로 조건에 영향받는 매개변수 기술
* 시스템 상태에 어떤 변화를 가져오는 **부작용**도 기술 ex) 백그라운드 스레드를 시작시키는 메서드
* 메서드의 계약(contract) 잘 지키려면 아래 태그를 달면 된다.
  * 모든 매개변수에 @param 태그 달기
  * 반환타입이 void가 아니라면 @return 태그 달기
  * 발생 가능성있는 모든 예외에 @throws 태그 달기
* 자바독 유틸리티는 문서화 주석을 html로 변환하므로 **html요소 사용 가능** ex) \<p>, \<i> 태그 등

## 종류별 태그 설명

**{@code}**

* @thorws 절에 사용
* 태그로 감싼 내용을 코드용 폰트로 렌더링
* 태그로 감싼 내용에 포함된 html요소, 자바독 태그 무시
* 여러줄로 된 코드를 넣으려면 \<pre>{@code …. 코드 …}\</pre> 형태로 작성해 줄바꿈을 유지할 수 있다.
* @기호에는 탈출문자를 붙여야 하므로 어노테이션 사용 시 주의

**@impleSpec**

* 자기사용 패턴
* 해당 메서드와 하위클래스 사이의 계약을 설명
* 하위클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출 시 메서드의 동작 설명

```javadoc
public interface List<E> extends Collection<E> {
     /**
     * Returns true if this list contains no elements.
     * @implSpec
     * This implementation returns {@code this.size == 0}.
     *
     * @return true if this list contains no elements
     */
     boolean isEmpty();
}
```

**{@literal}**

* <, >, & 등 HTML 메타문자를 포함시킬 수 있다.
* HTML 마크업이나 자바독 태그를 무시하게 한다.
* 코드 폰트로 렌더링은 하지 않는다.

```java
* {@literal |r| < 1} 이면 기하 수열이 수렴한다.
```

## 문서화 주석의 가독성

### 요약 설명

* 문서화 주석의 첫 문장은 해당 요소의 요약 설명이다.
* 한 클래스(인터페이스)안에 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안된다.
* 요약 설명에서는 마침표를 포함한 텍스트를 {@literal} 로 감싸야 한다. 요약 설명이 끝나는 기준이 (마침표+공백+대문자) 이기 때문이다.
* 자바 10 이상이라면 요약 설명에 {@Summary}를 사용하자.
* 메서드와 생성자의 요약설명은 해당 메서드와 생성자의 동작을 설명하는 ‘동사구’여야 한다.
  * ArrayList(int initialCapacity): Constructs an empty list with the specified initial capacity.
  * Collection.size(): Returns the number of elements in this collection.
* 클래스, 인터페이스, 필드의 요약설명은 ‘명사절’이어야 한다.
  * Instant: An instantaneous point on the time-line.
  * Math.PI: The double value that is closer than any other to pi, the ratio of the circumference of a circle to its diameter.

### 검색 기능

* 자바독이 생성한 HTML 문서에 검색 기능을 활용할 수 있다.
* 자바 9 이상부터 사용 가능
* API 문서 페이지 오른쪽 위에 있는 검색창에 키워드를 입력하면 관련 페이지들이 드롭다운으로 나타남
* {@index} 태그를 사용해 중요한 용어를 색인화 가능

## 제네릭, 열거, 어노테이션의 문서화

* 제네릭타입이나 제네릭 메서드는 모든 타입 매개변수에 주석을 달아야 한다.

```java
/**
* @param <K> the type of keys maintained by this map
* @param <V> the type of mapped values
*/
public interface Map<K, V> { }
```

* 열거타입 문서화 시 상수들에도 주석을 달아야 한다.

```java
public enum OrchestraSection {
/** Woodwinds, such as flute, clarinet, and oboe */
    WOODWIND,
/** Brass instruments, such as french horn and trumpet */
    BRASS,
/** Percussion instruments, such as timpani and cymbals */
    PERCUSSION,
/** Stringed instruments, such as violin and cello */
    STRING;
}
```

* 어노테이션 타입 문서화 시 멤버들에 모두 주석을 달아야 한다.

```java
/** // 어노테이션이 어떤 의미인지 설명 - 동사구
* Indicates that the annotated method is a test method that
* must throw the designated exception to pass.
*/@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
/** // 필드설명 - 명사구
    * The exception that the annotated test method must throw
    * in order to pass. (The test is permitted to throw any
    * subtype of the type described by this class object.)
    */
    Class<? extends Throwable> value;
}
```

## 문서화 주석 파일

* 패키지 설명은 package-info.java 에 작성한다.
* 모듈관련 설명은 module-info.java 에 작성한다.

## API 문서화에서 자주 누락되는 설명

* 스레드 안전성
  * 스레드 안전 수준을 반드시 API설명에 포함해야 함 (item 82)
* 직렬화 가능성
  * 직렬화 가능 클래스라면 직렬화 형태도 설명해야 함 (item 87)

## 자바독은 메서드 주석을 '상속'시킬 수 있다.

* 문서화 주석이 없는 API 요소를 발견하면 자바독이 가장 가까운 문서화 주석을 찾아줌
* 이때 상위 '클래스'보다 그 클래스가 구현한 '인터페이스'를 먼저 찾는다.
* {@inheritDoc} 태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수도 있다.

## 자바독 문서가 올바른지 확인하는 법

* 명령줄에서 -XdoClint 활성화 (자바7)
* 기본 제공 (자바8)
* checkstyle 같은 IDE 플러그인 사용
* HTML 유효성 검사기로 HTML 파일 돌리기
