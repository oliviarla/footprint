# 4장: 객체 구성

## 스레드 안전한 클래스

#### 조건

* 객체의 상태를 보관하는 변수가 어떤 것인지, 객체의 상태를 보관하는 변수가 어떤 값을 가질 수 있는지, 동시에 객체 내부의 값을 사용하고자 할 때 어떻게 관리하는지를 고려해야 한다.
* 객체 내부에 또다른 객체를 변수로 갖고 있는 경우, 객체의 상태 범위가 다양해지므로 주의해야 한다.
  * 예를 들어 LinkedList 객체의 상태 범위는 객체 내에 있는 모든 원소의 상태 범위를 포함한다.
* 객체의 상태 범위가 좁을수록 논리적인 상태를 파악하기 쉽다.
* 클래스가 특정 상태가 될 수 없도록 구현해야 한다면 그와 관련된 변수들을 클래스 내부에 숨겨두어야 한다.
* 특정 연산을 실행했을 때 올바르지 않은 상태 값을 가질 수 있다면 해당 연산을 단일 연산으로 구현해야 한다.
* 서로 연관된 변수들에 대해서 단일 연산으로 한 번에 읽거나 변경해야 한다.
* 현재 상태에 따라 동작 여부가 달라지는 상태 의존 연산을 수행할 때 특정 조건이 될 때 까지 기다려야 한다면 세마포어나 블로킹 큐를 사용하는 것이 좋다. wait, notify를 사용할 수도 있긴 하지만 올바르게 사용하기 어렵다.

#### 소유권

* 자바 언어 특성 상 소유권을 명확하게 표현하기 어렵지만, 객체의 내부에 상태 정보를 숨기면 객체의 상태에 대한 소유권을 가질 수 있다.
* 특정 변수를 객체 외부에 공개한다면 통제권을 어느 정도 잃게 된다.
* List, Set 등과 같은 컬렉션 클래스에서는 컬렉션 내부 구조에 대한 소유권은 컬렉션 클래스가 갖고, 컬렉션에 추가된 객체에 대한 소유권은 컬렉션을 사용하는 쪽에서 갖게 된다.
* 예를 들어 ServletContext 클래스는 Map과 유사한 구조로 만들어져 있으며 setAttribute/getAttribute 메서드를 통해 외부에서 생성된 객체를 추가하고 조회할 수 있다. 이 때 추가된 객체는 소유권이 ServletContext 객체에 없기 때문에 스레드 안전성을 충분히 확보해야 한다.

## 인스턴스 한정

#### 개념

* 데이터를 객체 내부에 캡슐화하여 숨겨두어 인스턴스에 한정되도록 하면, 해당 데이터는 당연히 객체 내부에서만 사용되므로 스레드 안전성을 확보하기 쉽다.
* 데이터를 객체 내부에만 숨겨둘 수도 있고, 특정 블록 내부에 한정시킬 수도 있고, 특정 스레드만 사용하도록 한정시킬 수도 있다. 이렇게 한정된 데이터는 다른 범위에 노출되면 안된다.
* `Collections.synchronizedList` 같은 팩토리 메서드에서 제공하는 wrapper 클래스는 데코레이터 패턴을 이용해 모든 메서드를 동기화시킨다.
* 특정 코드 범위 등에 한정되어야 할 객체가 외부에 공개된다면 버그가 발생할 수 있다.

#### 자바 모니터 패턴과 함께 인스턴스 한정 활용하기

* 변경 가능한 데이터를 모두 객체 내부적으로만 사용할 수 있도록 하고 객체의 암묵적 락을 통해 데이터에 대한 동시 접근을 막아 간결하게 스레드 안정성을 확보할 수 있다.
* 다음 예제 코드와 같이 내부 락을 `private final`로 선언하고 자바 모니터를 통해 제공되는 `synchronized` 블록으로 동시성 제어를 할 수 있다.

```java
public class PrivateLock {
    private final Object myLock = new Object();
    @GuardedBy("myLock") Widget widget;
    
    void doWithWidget() {
        synchronized(myLock) {
            // widget 변수 값을 읽거나 변경
        }
    }
}
```

## 스레드 안전성 위임

#### 위임 기법 활용

* 대부분의 자바 객체는 여러 객체를 조합해 만든 합성(composite) 객체이다.
* 합성 객체에서 가지는 여러 객체들이 스레드 안전성을 이미 확보하고 있고, 합성 객체가 가지는 상태가 해당 객체들 외에 따로 없다면 스레드 안전성을 해당 객체들에 위임(delegate)할 수 있다.
* 다음과 같이 불변 클래스인 `Point`를 포함하는 `ConcurrentHashMap`에 상태를 저장하는 경우, 별다른 동기화 작업을 하지 않고 `ConcurrentHashMap`에 그대로 스레드 안전성을 위임하게 된다.
* 외부에 Point 객체가 노출되더라도 외부에서 변경할 수 없으며, UnmodifiableMap으로 반환되므로 Map 자체도 외부에서 변경할 수 없다.
* 다만 UnmodifiableMap 내부의 데이터는 locations에  의해 변경 가능하다.

```java
public class DelegatingVehicleTracker {
    private final ConcurrentMap<String, Point> locations;
    private final Map<String, Point> unmodifiableMap;

    public DelegatingVehicleTracker(Map<String, Point> points) {
        locations = new ConcurrentHashMap<String, Point>(points);
        unmodifiableMap = Collections.unmodifiableMap(locations);
    }

    public Map<String, Point> getLocations() {
        return unmodifiableMap;
    }
        
    public Point getLocation(String id) {
        return locations.get(id);
    }
    
    public void setLocation(String id, int x, int y) {
        if (locations.replace(id, new Point(x, y)) == null) {
            throw new IllegalArgumentException();
        }
    }
}
```

#### 독립 상태 변수

* 클래스에 속한 변수가 여러 개이고 모두 스레드 안전하며 서로 상태 값에 대한 연관성이 없는 경우 스레드 안전성을 위임할 수 있다.
* 따라서 클래스가 별도로 동기화 작업을 추가하지 않아도 된다.
* 아래 예시 코드는 CopyOnWriteArrayList를 사용하여 스레드 안전하게 상태를 객체에 저장하고 있으므로, 별도로 클래스 단에서 동기화해주지 않고 있다.

```java
public class VisualComponent {
    private final List<KeyListener> keyListeners = new CopyOnWriteArrayList<KeyListener>();
    private final List<MouseListener> mouseListeners = new CopyOnWriteArrayList<MouseListener>();
    
    public void addKeyListener(KeyListener listener) {
        keyListeners.add(listener);
    }
    
    public void removeKeyListener(KeyListener listener) {
        keyListeners.remove(listener);
    }
    
    // ...
}
```

#### 위임이 불가능한 경우

* 클래스에 두 개 이상의 변수를 사용하는 복합 연산 메서드를 갖고 있다면, 각 변수가 스레드 안전하더라도 위임 기법으로 스레드 안전성을 확보할 수 없다.
* 따라서 변수 주변에 락을 적용하는 등 동기화 작업이 필요하다.
* 다음과 같이 setLower, setUpper 메서드에서 여러 변수를 참조하여 상태를 변경하는 경우 위임이 불가능하다. 두 메서드가 동시에 여러 스레드로부터 호출되는 경우 동기화가 되지 않고 있기 때문이다.&#x20;

```java
public class NumberRange {
    private final AtomicInteger lower = new AtomicInteger(0);
    private final AtomicInteger upper = new AtomicInteger(0);
    
    public void setLower(int i) {
        if (i < upper.get())
            throw new IllegalArgumentException();
        lower.set(i);
    }
    
    public void setUpper(int i) {
        if (i < lower.get())
            throw new IllegalArgumentException();
        upper.set(i);
    }
    
    // ...
}
```

#### 내부 변수를 외부에 공개

* 상태를 저장하는 변수가 스레드 안전하고 클래스 내부에서 상태 변수 값에 대해 의존성을 갖지 않는다면, 외부에서 어떤 연산을 수행하더라도 잘못된 상태가 될 일이 없으므로 공개해도 된다.

## 스레드 안전 클래스에 기능 추가

* 스레드 안전하게 구현된 클래스에 더 필요한 기능이 있는 상황이라면, 기존 클래스에 메서드를 추가하거나 클래스를 상속받아 하위 클래스에서 구현하면 된다.
* 하위 클래스에서 구현하는 경우 상위 클래스의 동기화 방식이 달라지는 경우 적절히 수정해주어야 한다.

#### 도우미 클래스

* 앞서 다룬 방식들을 사용할 수 없다면 도우미 클래스를 통해 추가 기능을 구현할 수 있다. 이 때 클라이언트 측 락 혹은 외부 락을 통해 내부 클래스가 도우미 클래스와 동일한 락을 사용하도록 하여 스레드 안전성을 확보해야 한다.
* 아래 코드는 클라이언트 측 락을 사용해 새로운 기능을 스레드 안전하게 추가한 예시이다.

```java
public class ListHelper<E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());
    
    // ...
    
    public boolean putIfAbsent(E x) {
        synchronized (list) {
            boolean absent = !list.contains(x);
            if (absent)
                list.add(x);
            return absent;
        }
    }
}
```

#### 클래스 재구성 (composition)

* 원본 클래스를 감싸는 클래스를 생성한 후, 원본 클래스가 갖고 있던 기능들을 새로운 메서드에서 호출하도록 하는 방식이다.
* 원래 클래스와 다른 수준에서 락을 활용하므로 원래 클래스가 스레드 안전한지 아닌지, 동기화 정책이 바뀌는지 등에 영향을 받지 않는다.
* 동기화 단계가 하나 더 추가되므로 전체적인 성능이 떨어질 수는 있다.

```java
public class ImprovedList<T> implements List<T> {
    private final List<T> list;
    
    public synchronized boolean putIfAbsent(E x) {
        boolean absent = !list.contains(x);
        if (absent)
            list.add(x);
        return absent;
    }
    
    public synchronized void clear() {
        list.clear(); // 원래 클래스의 메서드를 그대로 호출하면서 동기화 정책만 적용함
    }
    
    // ...
}
```

## 동기화 정책 문서화

* 클래스가 스레드에 안전한지, 락이 걸린 상태에서 콜백 함수를 호출하면 어떻게 되는지, 클래스의 동작 내용이 달라지는 락이 있는지 등 어디까지 스레드 안전성을 보장하는지 문서화해두어야 한다.
* 객체를 사용하는 입장에서 코드를 충분히 알고 있다는 가정을 하면 안된다.
* 외부 클래스에 저장하는 객체의 경우, 스레드 안전하거나 결론적으로 불변인 객체를 저장하는 것이 좋다.
