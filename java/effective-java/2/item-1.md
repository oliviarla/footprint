# item 1) 생성자 대신 정적 팩토리 메서드를 고려하라

## 정적 팩토리 메서드

* static factory method
* 직접 생성자를 호출해 객체를 생성하는 것이 아닌 정적 메서드를 통해서 객체를 생성하는 것

```java
public static MyObject create() {
    return new MyObject();
}
```

## **정적 팩토리 메서드의 장점**

### **1) 이름을 가질 수 있다**

* 생성자에 넘기는 매개변수와 생성자 자체만으로 반환될 객체의 특성을 쉽게 설명할 수 없다.
* 정적 팩토리 메서드는 한 클래스에 시그니처가 같은 생성자 여러 개가 필요할 때 각각 이름으로 구분지을 수 있다.
* 각 정적 팩토리 메서드의 역할을 쉽게 파악할 수 있도록 한다.

### **2) 호출될 때 마다 인스턴스를 새로 생성하지 않아도 된다**

* 불변 클래스의 경우 인스턴스를 미리 만들어두거나 새로 생성한 인스턴스를 캐싱해 재활용하여 불필요한 객체 생성을 피할 수 있다
  * ex) Boolean.valueOf(boolean) 메소드는 객체를 아예 생성하지 않는다
* 생성 비용이 큰 객체가 자주 요청될 때 성능 개선
* 인스턴스 통제 클래스이다.
  * 언제 어느 인스턴스를 살아있게 할 지 통제 가능
  * 클래스를 싱글턴, 인스턴스화 불가 등 원하는 방식으로 만들 수 있다.
  * 불변 값 클래스에서 동치인 인스턴스가 하나뿐임을 보장할 수 있다.

### **3) 반환 타입의 하위 타입 객체를 반환할 수 있다**

* 반환할 객체의 클래스를 자유롭게 선택 가능
* 구현 클래스를 공개하지 않고도 객체를 반환할 수 있다
* 인터페이스에 public 정적 팩토리 메서드를 선언해 하나뿐인 인스턴스화 불가 클래스를 얻을 수 있도록 한다.

### **4) 입력 매개변수에 따라 매번 다른 클래스의 객체 반환할 수 있다**

* 반환 타입의 하위 타입이면 어떤 클래스의 객체이든 반환 가능
* 클라이언트는 하위 타입의 클래스를 몰라도 해당 클래스의 정적 팩토리 메소드 사용 가능

### **5) 정적 팩토리 메소드 작성하는 시점에는 반환할 객체의 클래스가 필요 없다**

* 이를 통해 서비스 제공자 프레임워크의 근간이 될 수 있다.

#### 서비스 제공자 프레임워크

* 제공자(provider): 서비스의 구현체
* 프레임워크: 구현체들을 클라이언트에 제공하는 것을 통제하는 역할
* 핵심 컴포넌트
  1. 서비스 인터페이스: 구현체의 동작 정의
  2. 제공자 등록 API: 제공자가 구현체를 등록할 때 사용 (빈으로 등록?)
  3. 서비스 접근 API: 클라이언트가 서비스 인스턴스를 얻을 때 사용
* 클라이언트: 서비스 접근 API 사용 시 원하는 구현체의 조건을 명시할 수 있음
  * ex)getBean(GameService.class)

#### 다양한 서비스 제공자 프레임워크 패턴

* 브리지 패턴: 서비스 접근 API는 공급자가 제공하는 것보다 풍부한 서비스 인터페이스를 클라이언트에 반환
* 의존 객체 주입 프레임워크
* ServiceLoader: 자바에서 제공하는 범용 서비스 제공자 프레임워크

## **정적 팩토리 메서드의 단점**

### **1) 정적 팩토리 메서드만 제공 시 하위 클래스 생성 불가**

* 상속을 위해서는 public, protected 생성자가 필요하다. (super)
* 하지만 정적 팩토리 메서드만 제공하는 것은 생성자를 private으로 두겠다는 의미이다.
* 따라서, 상속보다 컴포지션을 사용하도록 유도한다.
* 불변 타입으로 만드려면 제약을 지켜야 한다.

### **2) 프로그래머가 찾기 어렵다**

* API 설정에 명확히 드러나지 않아 사용자가 정적 팩토리 메서드 방식 클래스를 인스턴스화 할 방법을 알아내야 한다.
* 널리 알려진 규약을 따라 네이밍해야 사용자가 이해하기 쉽다.
  * `from()` : 매개변수 하나를 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
  * `of()` : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  * `valueOf()` : from과 of의 자세한 버전
  * `instance()` / `getInstance()`: 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않음
  * `create()` / `newInstance()`: 매개변수로 명시한 인스턴스를 매번 새롭게 생성해 반환
  * `getType()`: getInstance와 같으나 인스턴스를 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 사용한다. Type자리에는 팩토리 메서드가 반환할 객체의 타입을 입력해줌
  * `newType()`: newInstance와 같으나 인스턴스를 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 사용한다. Type자리에는 팩토리 메서드가 반환할 객체의 타입을 입력해줌
  * `Type()`: getType과 newType의 간결한 버전
