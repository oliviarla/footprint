# item 34) int 상수 대신 열거 타입을 사용하라

## 열거 패턴

* 자바에서 열거 타입을 지원하기 전에는 정수 상수를 한 묶음으로 선언해 사용하였다.
* 타입 안전을 보장할 수 없고, 표현력이 좋지 않다.
* 컴파일하면 값이 클라이언트 파일에 그대로 넘겨지기 때문에 상수의 값이 변경되면 클라이언트도 다시 컴파일 해야 한다.

```java
public static final int GREEN_APPLE = 0;
public static final int RED_APPLE = 1;
public static final int COMPANY_APPLE = 2;
...
```

## 열거 타입

* 완전한 형태의 클래스이다.
* 상수 하나 당 인스턴스를 하나씩만 만들어 public static final 필드로 공개한다.
* 인스턴스를 하나만 만들기 때문에 싱글톤을 일반화한 형태라고 볼 수 있다.
* 컴파일타임 타입 안전성을 제공하여 타입이 다른 변수를 할당하려 할 때 컴파일 오류가 발생한다.
* 필요한 원소를 컴파일타임에 전부 알 수 있는 상수 집합일 때 사용하도록 한다.
* 추후 상수가 추가되어도 바이너리 수준에서 호환되므로 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

## 열거 타입과 메서드, 필드

* 특정 데이터와 열거 타입 상수를 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.
* 열거 타입에 선언한 상수 하나를 제거하더라도 **제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다**. 제거된 상수를 참조하면 컴파일 에러가 발생할 것이다.

### 데이터와 메서드를 갖는 열거 타입

* 열거 타입에는 임의의 메소드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
* 아래는 생성자에서 데이터를 받아 인스턴스 필드에 특정 데이터를 저장하여 연결한 예제이다.
* 열거 타입은 불변이므로, 모든 필드는 final이어야 한다.
* 최적화를 위해 surfaceGravity를 미리 계산해 저장해둘 수도 있다.

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6);

    private final double mass;
    private final double radius;
    private final double surfaceGravity;

    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
    	this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```

* values() : 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드
* 열거 타입에서 상수를 제거하는 경우, 클라이언트 프로그램을 다시 컴파일하면 컴파일 에러가 발생하고, 다시 컴파일하지 않으면 런타임 예외가 발생하여 정수 열거 패턴에 비해 훨씬 쉽게 디버깅할 수 있다.

### 상수별 메서드 구현

* 상수마다 동작이 달라져야 하는 상황일 때 사용한다.
* 하나의 메서드를 사용하는 것이 아닌, **열거 타입에 추상 메서드를 선언하고 각 상수에서 재정의**하도록 한다.

```arduino
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    public abstract double apply(double x, double y);

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

### 열거 타입 용 fromString 메서드

* toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 메서드
* Optional로 반환하여 값이 존재하지않을 상황을 클라이언트에게 알린다.

```tsx
private static final Map<String, Operation> stringToEnum =
		Stream.of(values())
                    .collect(toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

* stringToEnum을 생성자 외부에서 초기화하는 이유는 열거 타입에서 **정적 필드가 열거 타입 상수를 생성한 후 초기화** 되기 때문이다. 다시 말하자면 열거 타입 생성자에서 정적 필드인 stringToEnum을 참조해 본인 인스턴스를 추가하려고 하면, 아직 stringToEnum이 초기화되지 않은 상태이기 때문에 컴파일 에러가 발생할 것이다.

```java
private static final Map<String, Operation> stringToEnum = new HashMap<>();

Operation(String symbol) {
    this.symbol = symbol;
    stringToEnum.put(symbol, this); // 예외 발생!
}
```

## 전략 열거 타입 패턴

* 상수별 메서드 구현 시 열거 타입 상수끼리 코드를 공유하기 어렵다.
* switch문으로 상수별 사용할 로직을 다르게 지정할 수 있지만, 새로운 상수를 추가하면 메서드를 재정의해야 한다는 문제가 있다.
* 새로운 상수를 추가할 때 마다 **전략**을 선택하도록 한다. 전략 열거 타입을 하위에 지정하고 필요한 로직을 위임하여 사용한다.
* swithch문이나 상수별 메서드 구현이 필요 없게 되고, switch 문보다 복잡하지만 더 안전하고 유연하다.
* 아래의 예제는 잔업 수당 계산하는 로직을 private 전략 열거 타입에 두고, PayrollDay 열거 타입이 잔업수당 계산 필요 시 전략 열거 타입이 로직을 수행한다.

```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

* 다만, 기존 열거 타입에 상수별 동작을 혼합해 넣을 때에는 switch 문이 좋다.

```cpp
public double inverse(double x, double y) {
    switch(this) {
        case PLUS: return Operation.MINUS;
        case MINUS: return Operation.PLUS;
        case TIMES: return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;
    }
    throw new AssertionError("알 수 없는 연산: " + op);
}
```
