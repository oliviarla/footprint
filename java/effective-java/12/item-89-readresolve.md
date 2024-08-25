# item 89) 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

* 자바에서 싱글톤 객체의 직렬화/역직렬화를 제공하기 위해 `Serializable` 인터페이스를 구현하면 직렬화와 역직렬화 과정에서 싱글톤이 깨지게 된다.
* 왜냐하면 역직렬화 과정에서 어떤 readObject 메서드를 사용해도 **클래스가 초기화될 때의 인스턴스와 다른 인스턴스가 반환**되기 때문이다.

### readResolve 메서드

* 역직렬화한 객체의 클래스를 적절히 정의해두었다면, 새로 생성된 객체를 인수로 이 메서드가 호출되어, 새로 생성된 객체를 반환하지 않고 readResolve로부터 얻은 객체 참조를 반환한다.

```java
public class Singleton implements Serializable {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }

    // readResolve 메서드를 추가
    protected Object readResolve() {
        return INSTANCE;
    }
}
```

* 앞서 싱글톤 유지를 위해 readResolve 메서드를 정의한 것과 같이 인스턴스 통제 목적으로 이 메서드를 사용한다면, **모든 객체 필드를 transient로 선언**해야 한다.
* 만약 그렇지 않다면 item 88의 가변 공격 방식과 비슷하게 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 훔쳐올 수 있다.

{% hint style="info" %}
💡 자세한 과정

1. readResolve 메서드와 인스턴스 필드 하나를 포함한 **도둑 클래스**를 만든다. 도둑 클래스의 인스턴스 필드는 직렬화된 싱글턴을 참조한다.
2. 직렬화된 스트림에서 싱글턴의 non-volatile 필드를 도둑의 인스턴스 필드로 교체한다.
3. 싱글턴이 도둑을 참조하므로 역직렬화시 도둑 클래스의 readResolve가 먼저 호출된다.
4. 도둑 클래스의 인스턴스 필드에는 역직렬화 도중의 싱글턴의 참조가 담겨있게 된다.
5. 도둑 클래스의 readResolve 메서드는 인스턴스 필드가 참조한 값을 정적 필드로 복사한다.
6. 싱글턴은 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다.
7. 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 ClassCastException 이 발생한다.
{% endhint %}

```java
public class Singleton implements Serializable {
    public static final Singleton INSTANCE = new Singleton();
    private Singleton() {}
    private String[] names = {"hello", "world"};
    
    public void print() {
        System.out.println(Arrays.toString(names));
    }
    private Object readResolve() {
        return INSTANCE;
    }
}
```

```java
public class Stealer implements Serializable {
    static Singleton fakeSingleton;
    private Singleton singletonPayload;
    
    private Object readResolve() {
        fakeSingleton = singletonPayload;
        return new String[] {"April's Fool!"};
    }
    
    private static final long serialVersionUID = 0;
}
```

```java
byte[] manipulated = {...};

Singleton singleton;
try {
  singleton = (Singleton) new ObjectInputStream(
      new ByteArrayInputStream(manipulated)).readObject();
} catch (IOException | ClassNotFoundException e) {
  throw new IllegalArgumentException(e);
}

Singleton fake = Stealer.fakeSingleton;

singleton.print(); // hello world
fake.print(); // April's Fool!
```

### Enum

* 필드를 transient로 선언하여 문제를 해결할 수 도 있지만, 클래스를 원소 하나짜리 열거 타입으로 바꾸는 게 더 좋은 해결 방법이다.
* 열거 타입을 사용하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다. (AccessibleObject.setAccessible 같은 특권 메서드는 제외)

```java
public enum Singleton {
    UNIQUE_INSTANCE;
    
    public void do() {
        
    }
}

public class SingletonClient {
    public doSomething() {
        Singleton singleton = Singleton.UNIQUE_INSTANCE;
        singleton.do();
    }
}
```

### readResolve 메서드 주의점

* 컴파일타임에 어떤 인스턴스가 있는지 알 수 없는 상황이라면, Enum을 사용할 수 없으므로 readResolve 방식을 사용해야 한다.
* final 클래스에선 readResolve 메서드가 `private`이어야 한다.
* readResolve 메서드가 `protected/public` 이면서 하위 클래스에서 재정의 하지 않았다면, 하위 클래스의 인스턴스를 역직렬화할 때 상위 클래스의 인스턴스를 생성하므로 ClassCastException이 발생할 수 있다.
