# item 13) clone 재정의는 주의해서 진행하라

## Cloneable 인터페이스

* 복제해도 되는 클래스임을 명시하는 용도의 믹스인(mixin) 인터페이스(item 20)
* clone 메서드가 선언된 것이 Object이고, protected이므로 **Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다**.
* 해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없기 때문이다.
* Object의 protected 메서드인 clone의 동작 방식을 결정한다.
* Cloneable을 구현한 클래스의 인스턴스에서만 clone을 호출할 수 있다.

## clone 메서드의 일반 규약

* 이 규약은 허술하게 만들어져, 깨지기 쉽고 위험하고 모순적이다.
* 생성자를 호출하지 않고도 객체를 생성할 수 있다.

1. `x.clone() != x`  식은 참이다.
2. `x.clone().getClass() == x.getClass()` 식도 참이다. 하지만 반드시 만족해야 하는 것은 아니다.
3. `x.clone().getClass().equals(x)` 식도 일반적으로 참이지만 필수는 아니다.
4. `x.clone().getClass() == x.getClass()` 만약, super.clone() 을 호출해 얻은 객체를 clone()이 반환한다면 이 식은 참이다.

* 관례상 **반환된 객체와 원본객체는 독립적**이어야 한다.
* 이를 만족하기 위해 super.clone()으로 얻은 객체의 필드 중 하나 이상을 반환 전 수정해야 할 수도 있다. (이 내용은 아래 가변 객체 참조 클래스와 이어진다)
* final 클래스라면 하위 클래스를 가지지 못하므로 이 관례는 무시해도 된다.
* 불변 클래스의 경우는 clone 메서드 사용 시 쓸데없는 복사가 발생하기 때문에 굳이 제공하지 않는 것이 좋다.

## 가변 객체를 참조하는 클래스의 clone

* 단순히 super.clone() 을 반환하면 여러 복제된 인스턴스들이 같은 가변 객체를 참조하게 된다.
* 한 군데에서 수정하면 다른 부분에서도 수정된 효과가 나타나 이상 동작이나 NullPointerException이 발생할 수 있다.
* 이 경우 구현할 clone 메서드는 사실상 생성자와 같은 효과를 낸다.
* clone 메서드는 **원본 객체에 아무런 영향이 없도록 하며 복제된 객체의 불변식을 보장**해야 한다.

### 1) clone 메서드를 재귀적으로 호출

* 아래 예제에서는 result.elements의 **clone 메서드를 재귀적으로 호출**해주며 내부 정보를 복사한다.

```java
@Override
protected Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException cloneNotSupportedException) {
        throw new AssertionError();
    }
}
```

* 하지만, final로 선언된 가변 객체 필드는 위 방식을 사용할 수 없다. clone 메서드를 사용하려면 final 한정자를 제거해야할 수도 있따.

### 2) deep copy하는 clone 메서드 재정의

* 직접 구현된 클래스들의 배열이라면?
* 복제본은 자신만의 배열을 가질수는 있으나, 원본 객체의 배열 내부에 존재하는 객체를 참조하게 되므로 오동작할 수 있다.
* 하나하나 내부 정보를 반복자를 거쳐 deep copy해 배열을 다시 생성해야 한다.

```java
public class HashTable implements Cloneable{
    private Entry[] buckets = ...;
    ...
}
```

### 3) 원본 객체 상태를 생성하는 API 호출

* super.clone 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정 후, **원본 객체의 상태를 다시 생성(복제)하는 고수준 메서드**들을 호출하는 방식
* 저수준에서 바로 처리하는 것보다는 느리지만 코드가 깔끔하다.
* 고수준 메서드는 재정의될 수 없도록 final이거나 private이어야 한다.

## 주의할 것

* public인 clone 메서드는 예외를 throw하지 않도록 없애 편리하게 메서드를 사용하도록 한다.
* 상속용 클래스는 Cloneable을 구현하지 말고 clone 메서드를 protected로 재정의해 CloneNotSupportedException()을 던질 수 있도록 해야한다.

```java
protected Object clone() throws CloneNotSupportedException;
```

* 혹은 final로 선언해 하위 클래스에서 재정의 못하게 막을 수도 있다.

```java
@Override
protected final Object clone() throws CloneNotSupportedException {
	throw new CloneNotSupportedException();
}
```

* thread-safe 클래스를 작성하려면 clone 메서드를 재정의하고 동기화해야 한다.

## 복사 생성자/ 팩토리

* Cloneable/clone 방식보다 나은 면이 많다.
* 허술한 규약에 기대지 않고, final 필드를 사용해도 무방하며, 불필요한 검사 예외를 던지지 않고, 형변환도 필요 없다.
* 인터페이스 타입의 인스턴스를 인수로 받아 원본의 구현 타입에 관계없이 복제본의 타입을 선택할 수 있다. ex) HashSet 객체를 TreeSet 타입으로 복제할 수 있다.
* 인터페이스 기반 복사 생성자/팩토리를 **변환 생성자/팩토리**라고 부른다.

### 복사 생성자

* 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자

```cpp
public Yum(Yum yum){
	...
}
```

### 복사 팩토리

* 복사 생성자를 모방한 정적 팩토리

```tsx
public static Yum newInstance(Yum yum){
	...
}
```
