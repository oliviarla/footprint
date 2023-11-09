# item 79) 과도한 동기화는 피하라

## 과도한 동기화의 문제점

* 성능을 떨어뜨린다.
* 교착 상태에 빠뜨린다.
* 예측할 수 없는 동작이 발생할 수 있다.

### 동기화의 문제점

* 동기화 메서드나 블록 내에서는 제어를 클라이언트에 양도하면 안된다.
* 동기화된 영역 내에서는 재정의할 수 있는 메서드나, 클라이언트가 넘겨준 함수 객체를 호출하면 안된다.
* 해당 메서드가 어떤 일을 하는지 알지 못하며 통제할 수 없기 때문이다.
* 바깥 메서드들로 인해 예외를 일으키거나 교착상태에 빠지거나 데이터를 훼손할 수 있다.

## ObservableSet

* ForwardingSet을 상속받은 ObservableSet은 아래와 같다.

> ObservableSet은 간단히 말하면, 내부적으로 들고 있는 Set에 원소가 추가되면 등록되어 있던 Observer들에게 notify해주는 클래스이다.

```tsx
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();// 관찰자리스트 보관
    
    public void addObserver(SetObserver<E> observer) { // 관찰자 추가
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) { // 관찰자제거
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) { // Set에 add하면 관찰자의 added 메서드를 호출한다.
        synchronized (observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);// notifyElementAdded를 호출한다.
            return result;
    }
}

```

* addObserver, removeObserver의 인자로는 아래 콜백 인터페이스의 인스턴스를 건넨다.

```csharp
@FunctionalInterface
public interface SetObserver<E> {
    //ObservableSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```

### 재정의된 외부 메서드 사용 시 에러 발생

* addObserver를 통해 아래와 같이 직접 클라이언트가 작성한 함수 객체를 넘겨준다.

```csharp
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if(e == 23) {
                s.removeObserver(this);
            }
        }
    });

    // 재정의한 ObservableSet의 add 메서드 호출
    for(int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```

* 클라이언트가 직접 정의한 added 메서드 호출이 notifyElementAdded가 Observers의 리스트를 순회하는 도중에 발생하기 때문에, removeObserver가 수행되면서 ConcurrentModificationException이 발생한다.

> **로직의 흐름 간단 정리**
>
> 1. main에서 set.addObserver() 호출 -> ObservableSet의 List\<SetObserver>에 SetObserver 추가
> 2. for loop에서 ObservableSet의 add() 호출 -> notifyElementAdded() 호출
> 3. notifyElementAdded()에서 List\<SetObserver>를 순회하며, 클라이언트가 main에서 정의한 added()를 호출
> 4. added 메서드에서 조건이 만족되면 removeObserver() 를 호출
> 5. removeObserver()에서는 자신을 수정하는 것을 막지 못해 원소가 제거된다.
> 6. notifyElementAdded()에서 순회 중인 동기화 블록에서 ConcurrentModificationException이 발생한다.

### 재정의된 외부 메서드 사용 시 교착 상태 발생

* 동기화된 영역 내에서 외부 메서드를 호출하면 교착 상태에 빠질 수 있다.
* 아래 예제는 직접 removeObserver를 호출하지 않고, 실행자 서비스(ExecutorService)를 사용해 다른 스레드에게 부탁하는 방식을 사용한다.
* 실행자 서비스의 스레드는 락을 얻을 수 없어 removeObserver를 못하고, 메인 스레드는 백그라운드 스레드가 Observer를 제거하기를 기다리기 때문에 교착상태가 발생한다.

```csharp
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if(e == 23) {
            ExecutorService exec = Executors.newSingleThreadExecutor();
            try {
                exec.submit(() -> s.removeObserver(this)).get();// lock 걸림 - 접근 불가
                // 메인 스레드는 작업을 기다림
            } catch(ExecutionException | InterruptedException ex) {
                throw new AssertionError(ex);
            } finally {
                exec.shutdown();
            }
        }
    }
});

for(int i = 0; i < 100; i++) {
    set.add(i);
}
```

### 재정의된 외부 메서드 문제 해결 방법

* **동기화 블록 밖에서 외부 메서드를 호출**한다.
* 동기화 블록 밖에서 외부 메서드를 호출하는 것을 열린 호출이라고 부르며, 실패 방지 효과와 동시성 효율 개선의 효과가 있다.

```tsx
public void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }

    for (SetObserver<E> observer : snapshot) {
        observers.added(this, element);
    }
}
```

* **CopyOnWriteArrayList를 사용**한다.
* 항상 깨끗한 복사본을 만들어 수행하도록 구현해 락이 필요없으며 빠르다.

```csharp
private final List<SetObserser<E>> observers = new CopyOnWriteArrayList<>();

public void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers) {
        observers.added(this, element);
    }
}
```

* 이처럼 동기화 영역에서는 작업을 최소화해야 한다.
* Lock을 얻고, 공유 데이터를 검사하고, 필요하면 수정하고, Lock을 놓아야 한다.

## 동기화와 성능

* 병렬로 실행할 기회를 잃고, **모든 코어가 메모리를 일관되게 보기 위한 지연시간이 동기화의 진짜 비용**이다.
* 따라서 과도한 동기화를 피해야 한다.
* 가변 클래스 작성 시 아래의 두 개의 선택지 중 하나를 따라야 한다.
  * 동기화를 전혀 고려하지 말고, 사용하는 클래스가 외부에서 알아서 동기화하게 한다.
  * 동기화를 내부에서 수행해 thread-safe한 클래스로 만든다. (클라이언트가 외부에서 객체 전체에 락을 거는 것보다 효율적일 경우)
* Java의 라이브러리 중 java.util은 첫 번째 방법을, java.util.concurrent는 두 번째 방법을 택했다.
* 선택하기 어렵다면 동기화하지 말고 "스레드 안전하지 않다."라고 명시한다.

## 동시성을 높이는 기법

* 락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 사용할 수 있다.
* 락 분할(Lock Splitting)
  * 하나의 클래스에서 기능적으로 Lock을 분리해서 사용하는 것(읽기 전용 Lock, 쓰기 전용 Lock)
* 락 스트라이핑(Lock Striping)
  * 자료구조 관점에서 한 자료구조 전체가 아닌 일부분에 락을 적용하는 것
* 비차단 동시성 제어(NonBlocking Concurrency Control)
