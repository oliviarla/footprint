# item 10) equals는 일반 규약을 지켜 재정의 하자

## equals 재정의

* equals 메서드를 재정의 하지 않으면 해당 클래스의 인스턴스는 자기 자신과만 같게 된다.

### **equals 재정의가 필요 없는 경우**

* 값을 표현하는 게 아닌 동작하는 개체를 표현하여 인스턴스가 본질적으로 고유한 경우 (ex. Thread 클래스)
* 인스턴스의 논리적 동치성을 검사할 일이 없을 경우 (ex. Pattern 객체의 경우 여러 인스턴스가 같은 정규식을 표현하는지 검사할 때가 있음)
* 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞는 경우 (ex. 대부분의 Set, List 등의 구현체는 AbstractSet, AbstractList가 구현한 equals를 상속받아 그대로 사용)
* private, package-private 클래스이고 equals 메서드 호출하지 않는 경우
* enum과 같이 값이 같은 인스턴스가 둘 이상 만들어지지 않는 경우

### **equals 재정의가 필요한 경우**

* 객체 식별성이 아닌 논리적 동치성을 확인해야 할 때 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않을 때
* 값이 같을 때 객체도 같다고 정의해야 할 때

## equals 메서드의 일반 규약

### **반사성(reflexivity)**

* null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
* 객체는 자기 자신과 같아야 한다.

### **대칭성(symmetry)**

* null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
* ex) String을 갖는 임의의 CaseInsensitiveString 클래스의 equals 메소드에 다음과 같이 구현한다. String이 입력되면 대소문자 구분하지 않도록 하고 문자열이 같다면 true 반환한다. String의 equals()는 CaseInsensitiveString 클래스를 모르기 때문에, CaseInsensitiveString의 equals가 한 방향으로만 작동하게 되어 대칭성 위반한다.

### **추이성(transitivity)**

* null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
* 색상 값을 갖는 ColorPoint 클래스를 Point 클래스를 상속해 만들었다고 하자. ColorPoint와 Point 비교 시, Point는 Color 값이 없어 이 정보를 무시하고 위치만 같으면 equals에서 true를 반환하는 경우라면? 같은 위치의 다른 Color값을 가진 여러 ColorPoint가 같다고 판단되는 경우가 생긴다.
* 구체 클래스를 확장해 새로운 값을 추가하는 경우, equals의 일반 규약을 만족시킬 수 없다.
* 상속 대신 컴포지션을 사용(item 18) 해 규약을 지킬 수 있다.

### **일관성(consistency)**

* null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
* 두 객체가 같다면 영원히 같아야 한다.
* equals의 판단에 신뢰할 수 없는 자원(ex. host의 ip 주소처럼 상황에 따라 변경되는 정보) 을 사용하면 안된다.
* 항상 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

### **null-아님**

* null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.
* instanceof 연산자를 사용하면 잘못된 타입이 인수로 주어졌을 때 발생하는 ClassCastException 에러 방지 및 **null 타입 확인의 효과**가 있다.

## equals 메서드의 구현 방법

1. \==연산자를 사용해 입력이 자기 자신의 참조인지 확인
2. instanceof 연산자를 사용해 입력이 올바른 타입인지 확인
3. 입력을 올바른 타입으로 형변환
4. 입력 객체와 자기 자신의 대응되는 **핵심 필드들이 모두 일치하는지 검사**

> equals의 입력 인자는 반드시 Object객체여야 한다.

## 타입 별 비교 방법

* primitive type -> `== 연산자`로 비교
* float, double -> `compare 메서드`로 비교
* reference type -> `equals 메서드`로 비교
* 비교하기 복잡한 필드를 가진 클래스 -> `표준형을 정의`해 비교

#### **AutoValue**

클래스에 어노테이션을 추가하면 그 정보를 토대로 equals를 작성하고 테스트해주는 역할을 하는 프레임워크
