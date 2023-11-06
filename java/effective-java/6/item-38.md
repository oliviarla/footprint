# item 38) 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 확장 불가능한 열거 타입

* 타입 안전 열거 패턴은 확장할 수 있지만, 열거 타입은 확장이 불가능하다.
* 대개 열거 타입을 확장하는 것은 좋지 않다. 확장한 타입의 원소를 기반(부모) 타입의 원소로 취급하지만 그 반대는 성립하지 않는다.
* 기반 타입과 확장된 원소 모두를 순회할 방법도 마땅치 않다. 확장성을 높이면 고려할 요소가 늘어나 설계와 구현이 복잡해진다.

## 인터페이스를 사용해 열거 타입 확장하기

* 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다.
* 예를 들어 연산 기능을 하는 기본 열거 타입이 있고, 추가적으로 확장 연산 기능을 가진 확장 열거 타입을 구현해본다.
* 메서드를 정의한 인터페이스

```java
public interface Operation {
    double apply(double x, double y);
}
```

* 기본적인 연산 기능을 가지는 열거 타입

```tsx
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

* 추가적인 연산 기능을 가지도록 기본 타입을 확장한 열거 타입

```tsx
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

* 같은 인터페이스를 구현한 타입끼리는 서로 대체할 수 있다.

## 확장 열거 타입 사용해보기

### **방식 1**

* `T extends Enum<T> & Operation` 는 Enum 타입이면서 Operation의 하위 타입이어야 한다는 의미를 가진다.
* class 리터럴을 인자로 받도록 해 한정적 타입 토큰의 역할을 한다.

```cpp
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

### **방식 2**

* Class 객체 대신 한정적 와일드카드 타입을 인자로 받도록 한다.
* 여러 구현 타입의 연산을 조합해 호출할 수 있다.

```cpp
private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

## 인터페이스를 사용한 열거 타입 확장의 특징

* 열거 타입끼리 구현을 상속할 수 없다.
* 아무 상태에도 의존하지 않는 경우에는 디폴트 메서드 구현을 이용해 인터페이스에 추가하여 중복을 줄일 수 있다.
* 여러 확장된 공유하는 기능이 많을 경우 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다.
