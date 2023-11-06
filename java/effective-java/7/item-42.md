# item 42) 익명 클래스보다는 람다를 사용하라

## 함수 객체

* 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용했는데, 이 인터페이스의 인스턴스를 의미한다.
* 특정 함수나 동작을 나타내는 데 사용한다.
* 기존에는 주로 익명 클래스를 사용해 함수 객체를 만들었다.

```java
Collection.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2){
    return Integer.compare(s1.length(), s2.length());
  }
});
```

## 람다

* 간결하게 코드를 작성할 수 있어 어떤 동작을 하는지 명확히 드러난다.
* 컴파일러가 문맥을 살펴보고 타입을 추론해주므로 타입에 대한 언급이 없다. 컴파일러가 타입을 결정하지 못할 때에는 프로그래머가 직접 타입을 명시해야한다.
* 타입을 명시해야 코드가 명확해질 때를 제외하고는, 일단 매개 변수 타입을 생략할 것.

```java
Collections.sort(words,
	(s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

* 열거 타입의 인스턴스 필드를 이용하는 방식을 사용할 때 상수별로 다르게 동작하는 코드를 구현하기 쉽다.
* 아래 예제는 DoubleBinaryOperator 인터페이스 변수에 람다를 할당해 사용한다.

> DoubleBinaryOperator: Double 타입 인수 2개를 받아 Double 타입 결과를 돌려준다.

```java
public enum Operation {
    PLUS("+", (x, y) -> x+y),
    MINUS("-", (x, y) -> x-y),
    TIMES("*", (x, y) -> x*y),
    DIVDE("/", (x, y) -> x/y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y){
        return op.applyAsDouble(x, y);
    }
}
```

* 이름이 없고 문서화를 할 수 없다.
* 코드 자체로 동작이 명확히 설명되지 않거나 코드량이 많아지면 사용하지 말아야 한다.
* 되도록 1\~3줄 안에 끝나야 한다.
* 추상 클래스 인스턴스 만들 때, 추상메서드가 여러개인 인터페이스의 인스턴스를 만들 때에는 익명클래스를 사용해야 한다.
* 자신을 참조할 수 없으므로 함수 객체가 this로 자신을 참조해야 하면 익명클래스 사용해야 한다.
* vm마다 직렬화 형태가 다르기 때문에 직렬화 해야하는 함수 객체는 private 정적 중첩 클래스(Comparator처럼)의 인스턴스로 구현해야 한다. (익명클래스, 람다는 사용하지 말 것)
