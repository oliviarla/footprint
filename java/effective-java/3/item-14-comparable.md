# item 14) Comparable 구현을 고려하라

## Comparable 인터페이스

* Comparable 인터페이스를 구현한 객체들은 서로간의 비교 연산이 가능하다.
* compareTo 메서드는 equals와 달리 **단순 동치성 비교와 순서 비교가 가능하며 제네릭하다**.
* Comparable을 구현한 객체들의 배열은 다음처럼 손쉽게 정렬할 수 있다.

```java
Arrays.sort(a);
```

* Comparable을 구현하면 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션을 사용할 수 있다.
* 비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 TreeSet과 TreeMap이 있고, 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 Collections와 Arrays가 있다.
* 예를들어, String의 경우 Comparable을 구현했기 때문에 TreeSet에 입력되면 자동 정렬이 가능하다.
* Comparable이 구현되지 않은 객체라면? TreeSet에 넣어도 정렬이 되지 않을 것!
* 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 을 구현하자.

## compareTo 메서드의 일반 규약

* equals 메서드의 규약과 비슷하다.
* 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의정수를 같으면 0을, 크면 양의 정수를 반환한다.
* 비교할 수 없는 타입이 주어지면 대부분 ClassCastException을 던진다.
* 아래 규약들은 찬찬히 읽어보면 당연한 내용들이다.

1. 대칭성을 보장해야 한다.
   * Comparable을 구현한 클래스는 모든 x, y에 대하여 x.compareTo(y) == - y.compareTo(x) 여야 한다. 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다.
2. 추이성을 보장해야 한다.
   * (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0 이다.
3. 반사성을 보장해야 한다.
   * 모든 z에 대해 x.compareTo(y) == 0이면 x.compareTo(z) == y.compareTo(z)이다.
4. equals와 일관되어야 한다.
   * (x.compareTo(y) == 0) == (x.equals(y))이 되어야 한다. Comparable을 구현하고 이 권고를 지키지 않는 클래스는 그 사실을 다음 예시와 같이 명시해야 한다. (ex. 이 클래스의 순서는 equals 메서드와 일관되지 않습니다)

## compareTo 메서드 작성 요령

* equals와 달리 입력 인수의 타입을 확인하거나 형변환을 신경쓰지 않아도 된다.
* null이 입력되면 NullPointerException을 던져야 한다.
* compareTo 메서드는 각 필드 간의 순서를 비교하므로, 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다. (무슨말이지???)
* Comparable을 구현하지 않은 필드나 직접 순서를 지정해 비교해야 한다면 비교자(Comparator)를 대신 사용한다.
* 비교자는 직접 구현할수도 있고, 자바에서 제공하는 compare 메서드를 사용할 수도 있다.
* 기본 타입을 비교할 때에는, 박싱된 기본타입 클래스들에서 제공하는 compare 정적 메서드를 사용할 수 있다.
* 필드의 값을 비교할 때 <와> 연산자는 지양해야 한다.
* 클래스의 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교한다.
* 결과가 똑같지 않은 필드를 찾을 때 까지 비교해나가도록 구현하면 된다.

```kotlin
public int compare(final PhoneNumber phoneNumber) {
        int result = Short.compare(areaCode, phoneNumber.areaCode);

        if (result == 0) {
            result = Short.compare(prefix, phoneNumber.prefix);
            if (result == 0) {
                result = Short.compare(lineNum, phoneNumber.lineNum);
            }
        }

        return result;
}
```

* Comparable을 구현한 클래스를 확장하고 필드를 추가하는 경우 compareTo를 재정의할 수 없다. ⇒ Composition 활용

## Comparator

* 자바 8부터 일련의 비교자 생성 메서드와 메서드 연쇄방식으로 비교자를 생성할 수 있게 되었다.
* compareTo 메서드 구현에 활용할 수 있다.
* 아래 예제는 비교자 생성 메서드 두 가지(comparingInt, thenCompaingInt)를 이용해 비교자를 생성한다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
            Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	    return COMPARATOR.compare(this, pn);
}
```

> comparingInt(): 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받고, 그 키를 기준으로 순서를 정하는 비교자를 반환

* 수많은 보조 생성 메서드, 객체 참조용 비교자 생성 메서드들을 제공한다.

> comparing(): 1) 키 추출자를 받아 자연적 순서를 사용하거나, 2) 키 추출자와 추출된 키를 비교할 비교자를 입력받는다

## 비교자, 이렇게 쓰면 안된다

* int 값을 직접 뺄셈하면, overflow나 부동 소수점 계산 방식에 따라 오류가 발생할 수 있다.

```dart
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
};
```

* 가급적 아래와 같이 자바에서 제공되는 compare 메서드를 활용하도록 하자.

1. 정적 compare 메서드를 활용한 비교자

```dart
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
};
```

2. 정적 compare 메서드를 활용한 비교자

```jsx
static Comparator<Object> hashCodeOrder =
Comparator.comparingInt(o -> o.hashCode());
```
