---
description: Optional 사용 방법에 대해 알아본다.
---

# 11장: null 대신 Optional

## null의 문제점

### NullPointerException

* 코드에 아무런 문제가 없어보이더라도 null 참조를 반환하는 메서드에 체이닝된 메서드를 두는 등 null 값에 접근해 무언가 하려하면 NullPointerException이 발생할 수 있다.
* getCar() 메서드에서 null이 반환되면 getInsurance()를 실행할 수 없어 NullPointerException이 발생할 것이다.

```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

* 이를 해결하기 위해 모든 반환값을 null 체크할 수도 있지만 코드 구조나 가독성 측면에서 좋지 않다.

```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```

* null은 자바에서 가장 많이 발생하는 예외 중 하나를 발생시킨다.
* Null 체크를 빈번히 하게되면 코드가 지저분해진다.
* Null 은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로 적합하지 않다.
* 자바는 개발자로부터 모든 포인터를 숨겼지만 Null 포인터는 예외이다.
* Null은 무형식이며 정보를 포함하지 않고 있기 때문에 모든 참조 형식에 Null을 할당할 수 있어 형식 시스템에 구멍을 만든다.

### 다른 언어에서 null 대처법

* groovy의 경우 safe navigation operator(?.) 를 도입해 null이 할당되었다면 즉시 null을 반환한다.

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

## Optional 클래스

* Java 8은 하스켈과 스칼라의 영향을 받아 Optional이라는 클래스를 도입했다.
* Optional은 선택형 값을 캡슐화하는 클래스이다.
* 값이 있으면 Optional 클래스는 값을 감싸고, 값이 없으면 Optional.empty() 메서드로 값이 없는 Optional 객체를 반환한다.
* 아래와 같이 사람은 차를 소유할 수도 있고 소유하지 않을수도 있음을 Optional을 통해 명확하게 표현할 수 있다.

```java
public class Person {
    private Optional<Car> car;

    public Optional<Car> getCar() {
        return car;
    }
}
```

* Optional은 필드 형식으로 사용될 것을 가정하고 제공되는 것이 아니기 때문에 Serializable 인터페이스를 구현하지 않는다. 따라서 직렬화를 사용하는 도구나 프레임워크에서 문제가 생길 수 있다. 직렬화 모델이 필요하면 아래와 같이 Optional로 값을 반환받을 수 있는 메서드를 추가하는 것을 권장한다.

```java
public class Person {
    private Car car;

    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```

## Optional 적용 패턴

* Optional을 사용하면 명시적으로 형식 시스템을 정의할 수 있고, 메서드가 빈 값을 받거나 빈 결과를 반환할 수 있음을 문서화하는 것과 같은 역할을 한다.

### Optional 생성

```java
// 빈 옵셔널 생성
Optional<Car> optCar = Optional.empty();

// 값을 담은 옵셔널 생성
Optional<Car> optCar = Optional.of(car);

// nullable한 값을 담은 옵셔널 생성
Optional<Car> optCar = Optional.ofNullable(car);
```

### map 메서드

* map 메서드를 사용하여 값을 추출하고 변환할 수 있다.

```java
Optional<Long> price = Optional.ofNullable(car).map(Car::getPrice);
```

### flatMap 메서드

* Optional의 map 메서드를 사용하면 결과로 Optional 객체가 나온다.
* 따라서 아래와 같이 map 메서드를 체이닝하면 `Optional<Optional<Car>>` 처럼 중첩된 Optional이 반환되므로 컴파일되지 않는다.

```java
Optional<Person> optionalPerson = Optional.of(person);
Optional<String> name = optionalPerson.map(Person::getCar)
                                      .map(Car::getInsurance)
                                      .map(Insurance::getName);
```

* flatMap 메서드를 사용하면 map 메서드를 체이닝하더라도 여러 차원의 Optional을 하나의 Optional 객체로 flatten하여 반환한다.

```java
Optional<Person> optionalPerson = Optional.of(person);
String name = optionalPerson.flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("UNKNOWN");
```

### Optional 스트림 조작

* Optional은 stream() 메서드를 제공하여 파이프라인 동작 시에 Optional에 값이 있는 경우만 스트림으로 변환해준다.

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
            .map(Person::getCar) // Stream<Optional<Car>>
            .map(optCar -> optCar.flatMap(Car::getInsurance)) // Stream<Optional<Insurance>>
            .map(optInsurance -> optInsurance.map(Insurance::getName)) // Stream<Optional<String>>
            .flatMap(Optional::stream) // Stream<String>, Optional을 Unwrap하였음
            .collect(Collectors.toSet()); // Set<String>
}
```

### 값 다루기

* `get()`: Optional에서 값을 읽는 가장 간단한 메서드이지만 값이 없으면 NoSuchElementException이 발생하므로 값이 있다고 반드시 가정할 수 있는 상황이 아니라면 사용하지 않는 것이 좋다.
* `orElse(T other)`: Optional에 값이 없는 경우에 기본값을 반환할 수 있다.
* `orElseGet(Supplier<? extends T> other)`: Optional에 값이 없는 경우 Supplier가 실행된다. 기본값을 만들어내는 데에 수행 시간이 오래걸리거나 Optional이 비어있을 때만 기본값을 생성하고 싶을 때 사용한다.
* `orElseThrow(Supplier<? extends T> exceptionSupplier)`: Optional이 비어 있을때 예외를 발생시킨다는 점에서 get 메서드와 비슷하며, Supplier에서 발생시킬 종류의 예외를 반환하면 된다.
* `ifPresent(Consumer<? super T> consumer)`: 값이 존재할 때 인수로 넘겨준 동작을 실행한다.
* `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`: Java 9 버전에 추가되었으며, 값이 존재하면 action을 수행하고 값이 비어있을 때에는 Runnable을 수행한다.

### 두 Optional 합치기

* 두 Optional 정보를 이용해 값이 존재하는 경우에만 두 정보를 이용하도록 하려면 아래와 같이 작성할 수 있다.
* 다만 아래와 같이 구현하면 null 확인 코드와 다를바가 없다.

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (person.isPresent() && car.isPresent()) {
        return Optional.of(findCheapestInsurance(person.get(), car.get()));
    }
    return Optional.empty();
}
```

* flatMap과 map을 사용해 person과 car가 모두 존재하면 map 메서드에 의해 findCheapestInsurance가 호출되도록 할 수 있다.

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    return person.flatMap(p -> car.map(findCheapestInsurance(p, c)));
}
```

### 필터로 특정 값 거르기

* 객체의 프로퍼티 값이 일치하는지 확인하기 위해서 아래와 같이 filter 메서드로 확인할 수 있다.
* Optional 객체에 값이 존재하고 Predicate를 만족하면 해당 값을 반환하고, 그렇지 않으면 빈 Optional을 반환한다.

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "Hyundai".equals(insurance.getName()))
            .ifPresent(x -> ...);
```

## Optional 사용하기

### nullable한 변수 감싸기

* 기존 자바 API에서 null을 반환하여 요청한 값이 없거나 계산이 실패함을 알리는 것을 Optional로 감싸 안전하게 처리한다.

```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 예외 처리 시 유틸 메서드 사용

* 값이 없거나 처리할 수 없을 때 예외를 발생시키는 API를 래핑한 유틸 메서드를 만들어 예외 대신 Optional 형태로 반환하도록 할 수 있다.
* 아래는 Integer#parseInt 메서드에서 예외가 발생하면 빈 Optional을 반환하도록 하는 유틸 메서드이다.

```java
public static Optional<Integer> stringToInt(String s) {
    try {
        return Optional.of(Integer.parseInt(s));
    } catch (NumberForamtException e) {
        return Optional.empty();
    }
}
```

### 기본형 특화 Optional 자제

* Optional에서는 기본형 특화 Optional인 `OptionalInt, OptionalLong, OptionalDouble` 등의 클래스를 제공한다.
* 스트림의 경우에는 많은 요소를 가질 때 기본형 특화 스트림을 이용해 불필요한 boxing / unboxing을 줄여 성능을 향상시킬 수 있지만, Optional의 최대요소는 한개이므로 기본형 특화 클래스를 사용하더라도 차이가 없다.
* 더군다나 기본형 특화 Optional은 map, flatMap, filter등을 지원하지 않고 일반 Optional과 호환 되지 않아 스트림 파이프라인을 구성할 때 불편하다.
