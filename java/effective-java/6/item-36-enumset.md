# item 36) 비트 필드 대신 EnumSet을 사용하라

## 비트 필드 열거 상수

* 열거한 값들이 집합으로 사용될 경우 아래와 같이 정수 열거 패턴을 사용한다.
* 정수들을 비트 연산을 통해 정수 형태의 부분집합을 만들 수 있는데, 이를 비트 필드라고 부른다.
  * ex) 아래의 경우 bold와 italic을 사용하면 `0011`, italic과 highlight를 사용하면 `1010`

```java
public class Text {
    public static final int STYLE_BOLD      = 1<<0; //1
    public static final int STYLE_ITALIC    = 1<<1; //2
    public static final int STYLE_UNDERLINE = 1<<2; //4
    public static final int STYLE_HIGHLIGHT = 1<<3; //8
    
    public void applyStyles(int styles) { ...}
}
```

* `text.applyStyles(STYLE_BOLD | STYLE_ITALIC)` 과 같이 비트별 OR을 사용할 경우 쉽게 여러 상수들을 하나의 집합으로 모을 수 있다.
* 비트 필드를 사용하면 비트별 연산을 사용해 합집합, 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

### 단점

* 정수 열거 상수의 단점(item 34)을 그대로 갖는다.
* 비트 필드 값이 출력되면 단순한 정수 열거 상수를 출력하는 것 보다 해석이 어렵다.
* 비트 필드는 집합을 나타내는데, 이 집합에 어떤 원소들이 포함되어 있는지 순회하기 어렵다.
* 최대 몇 비트가 필요한지 API 작성 시에 미리 예측해 적절한 타입을 선택해야 하므로 변경에 용이하지 않다.

## EnumSet

* 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현한다.
* Set 인터페이스를 구현하고, type safe하다.
* 내부적으로 비트 벡터로 구현되어 원소가 64개 이하면 long 변수 하나로 모든 필드를 표현한다.

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STYLE_HIGHLIGHT }
    
    public void applyStyles(Set<Style> styles) { ...}
}
```

* `text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC))` 과 같이 사용할 수 있다.
