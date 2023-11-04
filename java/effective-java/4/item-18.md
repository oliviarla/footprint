# item 18) 상속보다는 컴포지션을 사용하라

## 상속의 문제점

* 클래스 상속을 사용하는 경우 코드를 재사용할 수 있다는 장점이 있지만, 다른 패키지의 구체 클래스를 상속할 때 위험을 초래한다.
* 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때 사용해야 한다.

### **캡슐화를 깨뜨리는 상속**

* 상위 클래스의 내부 구현이 변함에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
* 상위 클래스 설계자가 확장을 충분히 고려하고 문서화를 해두지 않는다면 하위 클래스는 상위 클래스의 변경에 따라 수정되어야 한다
* 예를 들어 아래와 같이 HashSet을 구현한 임의의 InstrumentedSet 클래스에서 addAll 메서드를 사용하여 addCount를 증가시킬 경우를 생각해본다. 만약 3개의 인자를 더한다면 addCount가 3이어야 하겠지만, HashSet의 addAll 메서드 내부에서 add 메서드를 사용(self-use)하고 있어 의도치 않게 두 번 더해지는 결과가 나온다.

```tsx
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll (Collection< ? extends E > c){
        addCount += c.size();
        return super.addAll(c);

    }
}
```

* 자신의 다른 부분을 사용하는 self-use 여부는 해당 클래스의 내부 구현 방식이며 전반적인 정책인지, 다음 릴리즈에 유지되는지 알 수 없다.

### **상위 클래스에 새로운 메서드를 추가했을 때**

* 상위 클래스의 다음 릴리즈에서 나온 새로운 메서드를 이용해 하위 클래스에서 '허용되지 않은' 원소를 추가하면 문제가 발생한다. (ex. Hashtable, Vector가 컬렉션 프레임워크에 포함되었을 때 보안 문제들을 수정해야 했다.)

### **하위클래스에 추가한 메서드가 상위 클래스 새 릴리즈에서 추가된 메서드와 같은 시그니처를 가지고 다른 반환 타입을 가질 때**

* 시그니처가 같고 반환 타입이 다르면 컴파일조차 불가능
* 이미 하위 클래스에서 정의되었기 때문에 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 가능성이 크다.

## 컴포지션 설계

* 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조한다.
* Forwarding 메서드를 구현한다.

> Forwarding 메서드: 기존 클래스에 대응하는 메서드를 호출해 결과를 반환하도록 새 클래스에 생성한 인스턴스 메서드

* 새 클래스는 기존 클래스의 내부 구현방식이나 새로운 메서드 추가에 영향받지 않는다.
* 컴포지션을 써야할 상황에서 상속을 사용하면 내부 구현에 묶이고 클래스의 성능도 영원히 제한되며 클라이언트가 노출된 내부에 직접 접근할 수 있다.
* 컴포지션은 상위 클래스의 결함을 숨기는 새로운 API를 설계할 수 있다.
* 아래는 상속 대신 컴포지션을 사용하는 예시이다.

### ForwardingSet 클래스

* 전달 메서드만으로 이뤄진 재사용 가능한 전달 클래스이다.
* 작성이 귀찮을 수 있으나 인터페이스 당 하나만 만들어두면 원하는 기능을 덧씌우는 클래스를 손쉽게 구현 가능하다.

```tsx
public class ForwardingSet<E> implements Set<E> {

    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s;}

    public void clear() { s.clear(); }
    public boolean contains(Object o) {return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s. iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) {return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override public boolean equals(Object o) { return s.equals(o); }
    @Override public int hashCode() {return s.hashCode(); }
    @Override public String toString() { return s.toString(); }

}
```

### InstrumentedSet 클래스

* HashSet의 모든 기능을 정의한 Set 인터페이스를 구현해 설계되었기에 견고하고 유연하다.
* Set의 인스턴스를 인수로 받는 생성자를 제공한다.
* 다른 인스턴스를 감싸고 있는 **래퍼 클래스**(wrapper class)이며, 다른 Set에 계측 기능을 덧씌웠기에 데코레이터 패턴이라고도 한다.

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll (Collection< ? extends E > c){
        addCount += c.size();
        return super.addAll(c);

    }

    public int getAddCount () {
        return addCount;
    }
}
```

* 래퍼 클래스는 단점이 거의 없지만, 자기 자신의 참조를 다른 객체에 넘겨 다음 호출에 사용하도록 하는 콜백 프레임워크와는 어울리지 않는다. 내부 객체는 자신을 감싸는 래퍼의 존재를 몰라 자신의 참조를 넘기고, 콜백 시 래퍼가 아닌 내부 객체를 호출하게 된다.
* **위임(delegation)**: composition과 forwarding의 조합이다. 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우 해당된다.
