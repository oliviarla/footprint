# 반복자 패턴

## 접근

* 객체를 모아둔 컬렉션 반복 작업 처리를 컬렉션 구현에 의존하지 않고 모두 동일하게 수행하고 싶은 경우, 반복하는 행위를 캡슐화하는 인터페이스를 두도록 한다.
* 예를 들어 리스트, 셋, 트리 등의 컬렉션을 어떻게 순회할 지는 반복자 구현체가 결정한다. 사용자는 반복자 구현체가 제공하는 메서드를 사용해 하나씩 꺼내오기만 하면 될 것이다.

## 개념

* 컬렉션의 구현 방법을 노출하지 않으면서 집합체 내의 모든 항목에 접근하는 방법을 제공하는 패턴이다.
* 모든 항목에 접근하는 작업을 컬렉션 객체가 아닌 반복자 객체가 맡게 된다.

<figure><img src="../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

## 장단점

* 장점
  * 반복자가 구현되어 있다면 어떤 컬렉션이든 동일한 순환문을 통해 처리할 수 있다.
  * 반복문이 필요한 부분에서는 반복자 인터페이스만 알면 된다.
  * 반복하는 부분에 변경이 필요하면 반복자만 변경하면 되므로 SRP(단일 역할 원칙)을 지킨다.
  * 클라이언트는 반복자 인터페이스에만 의존하므로 IRP(인터페이스 의존 법칙)을 지킨다.
  * 새로운 타입의 컬렉션들에 대해 반복자들을 구현할 수 있으며, 기존의 코드에 영향을 주지 않은 채 사용할 수 있기 때문에 OCP(개방 폐쇄 원칙)을 지킨다.
* 단점
  * 단순한 컬렉션들을 사용하는 경우 반복자 패턴이 과할 수 있다.
  * 일부 컬렉션들에서는 직접 탐색하는 것보다 비효율적일 수 있다. ArrayList의 경우 특정 위치에 쉽게 접근할 수 있지만 LinkedList의 경우 특정 노드를 저장해두어야만 하는 것이 그 예이다.

## 사용 방법

* Iterator 인터페이스를 정의하여 어떠한 컬렉션 구현체이든 반복할 수 있도록 한다.

```javascript
public interface Iterator {
    boolean hasNext();
    MenuItem next();
}
```

* Iterator 구현체에는 컬렉션과 현재까지의 반복 작업 처리 위치를 저장하고, next() 메서드로는 다음 원소를 제공하고, hasNext() 메서드로는 다음 원소가 있는지 여부를 제공한다.

```java
public class DinerMenuIterator implements Iterator {
    MenuItem[] items;
    int position = 0;
    
    public DinerMenuIterator(MenuItem[] items) {
        this.items = items;
    }
    
    public MenuItem next() {
        MenuItem menuItem = items[position];
        position += 1;
        return menuItem;
    }
    
    public boolean hasNext() {
        if (position >= items.length || items[position] == null) {
            return false;
        } else {
            return true;
        }
    }
}
```

## 예시

### 자바의 Iterable 인터페이스

* Collection 인터페이스는 Iterable 인터페이스를 상속받는다. 따라서 Collection 구현체들은 반복자(Iterator 구현체)를 반환하는 `iterator()` 메서드와 반복 작업을 내부적으로 수행하도록 하는 `forEach(Consumer<? super T> action)` 메서드를 제공한다.

```java
public interface Iterable<T> {

    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

```java
public interface Collection<E> extends Iterable<E> {

    int size();

    boolean isEmpty();
    
    // ...
}
```

### Spring의 InfoProperties

* 각종 프로퍼티를 저장하는 클래스에서 Iterable을 구현하여 iterator 메서드를 오버라이드했다.
* 내부에 있는 프로퍼티 데이터들은 변경되면 안되므로 Iterator의 remove 메서드는 UnsupportedOperationException 예외를 던지며 지원하지 않도록 했다.

```java
public class InfoProperties implements Iterable<InfoProperties.Entry> {
    private final Properties entries;
    
    @Override
    public Iterator<Entry> iterator() {
    	return new PropertiesIterator(this.entries);
    }
    
    private static final class PropertiesIterator implements Iterator<Entry> {

        private final Iterator<Map.Entry<Object, Object>> iterator;
        
        private PropertiesIterator(Properties properties) {
        	this.iterator = properties.entrySet().iterator();
        }
        
        @Override
        public boolean hasNext() {
        	return this.iterator.hasNext();
        }
        
        @Override
        public Entry next() {
        	Map.Entry<Object, Object> entry = this.iterator.next();
        	return new Entry((String) entry.getKey(), (String) entry.getValue());
        }
        
        @Override
        public void remove() {
        	throw new UnsupportedOperationException("InfoProperties are immutable.");
        }
    
    }
}
```

### Spring의 CompositeIterator

* Spring에서는 여러 Iterator 들을 등록하여 모든 Iterator의 아이템을 순회하도록 할 수 있는 클래스를 제공한다.
* 내부적으로 CompositeSet, CompositeMap 등을 구현할 때 사용된다.

```java
public class CompositeIterator<E> implements Iterator<E> {

    private final Set<Iterator<E>> iterators = new LinkedHashSet<>();
    private boolean inUse = false;
    
    public void add(Iterator<E> iterator) {
        Assert.state(!this.inUse, "You can no longer add iterators to a composite iterator that's already in use");
        if (this.iterators.contains(iterator)) {
            throw new IllegalArgumentException("You cannot add the same iterator twice");
        }
        this.iterators.add(iterator);
    }

    @Override
    public boolean hasNext() {
        this.inUse = true;
        for (Iterator<E> iterator : this.iterators) {
            if (iterator.hasNext()) {
                return true;
            }
        }
        return false;
    }
    
    @Override
    public E next() {
        this.inUse = true;
        for (Iterator<E> iterator : this.iterators) {
            if (iterator.hasNext()) {
                return iterator.next();
            }
        }
        throw new NoSuchElementException("All iterators exhausted");
    }
}
```
