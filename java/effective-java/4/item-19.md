# item 19) 상속을 고려해 설계하고 문서화하고, 그러지 않았다면 상속을 금지하라

## 상속을 허용하는 클래스가 지켜야 할 제약

### **문서화**

* 상속용 클래스는 재정의할 수 있는 메소드들을 내부적으로 어떻게 이용하는지(self-use) 문서로 남겨야 한다.
* 호출되는 메소드가 하위 클래스에서 재정의 가능한 메소드라면 이 사실을 호출하는 메소드의 API 설명에 명시해야한다.
* 재정의 가능 메소드를 호출할 수 있는 모든 상황을 문서로 남겨두어야 한다.
* @implSpec태그를 통해 내부 동작 방식을 설명하는 부분을 생성할 수 있다. 활성화 하기 위해선 javadoc -tag "implSpec:a:Implementation Requirements:"를 지정해줘야 한다.

### **protected 메서드의 제공**

* 효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 hook을 잘 선별해 protected 메서드 형태로 공개할 수 있다.
* 예를 들어 List의 removeRange 메소드는 해당 리스트 또는 부분 리스트의 clear 메소드에서 호출한다. 리스트 구현의 내부 구조의 이점을 잘 활용하여 removeRange메소드를 재정의하면 해당 리스트 또는 부분 리스트의 clear메소드 성능을 향상 시킬 수 있다. 이 메소드를 제공한 이유는 단지 **하위 클래스에서** removeRange 메서드를 **재정의해** clear 메소드를 **고성능으로 만들도록** 하기 위해서이다.
* 상속용 클래스에서 어떤 메소드를 protected로 노출해야 할지는 직접 하위 클래스를 만들어보아야 알 수 있다. 또한 상속용 클래스는 배포 전에 반드시 3개 정도의 하위 클래스를 만들어 검증해야한다.
* 하위 클래스를 여러 개 만들때까지 전혀 쓰이지 않는 protected 멤버는 사실 private이었어야 할 가능성이 큼

### **생성자에서 재정의 가능한 메서드 호출 금지**

* 상위 클래스의 생성자에서 하위 클래스에서 재정의한 메서드가 사용되는 경우 하위 클래스 생성자보다 하위 클래스의 재정의 메서드가 먼저 호출된다. 이 때 하위클래스의 재정의 메서드가 하위클래스 생성자에서 초기화하는 값에 의존하는 경우 오동작 한다.
* 아래와 같은 경우 하위 클래스의 재정의 메서드가 하위 클래스 생성자의 str 값에 의존하기 때문에, 생성자에서 재정의 가능 메서드 호출하면 null값이 들어가게 된다.

```java
public class Super {a
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
}
```

```java
public class Sub extends Super{
    private String str;
    public Sub() {
        str = "initialize Sub class";
    }

    @Override
    public void overrideMe() {
        System.out.println(str);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

<figure><img src="../../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

#### **Cloneable과 Serializable 인터페이스**

* Cloneable의 clone과 Serializable의 readObject는 새로운 객체를 만들기 때문에 생성자와 비슷한 효과를 낸다. 따라서 상속용 클래스에서 이 두가지 인터페이스를 구현하는 경우 생성자와 같은 제약을 따라야 한다. 결국, **clone과 readObject 모두 재정의 가능한 메서드를 호출하면 안된다.**
* readObject는 하위 클래스 상태가 역직렬화 되기 전에, clone은 복제본의 상태를 올바르게 수정하기 전에(원본 객체의 참조를 여전히 갖고 있는 경우), 재정의한 메서드를 호출할 수 있다.
* Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 가지면, protected로 선언해야 하위 클래스에서 무시되지 않는다.

### **구체 클래스의 상속**

* 구체 클래스의 내부만 수정해도 자식 클래스에서 문제가 발생하는 경우가 많다.
* 따라서 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 좋다.
* 혹은 구체 클래스가 핵심 기능을 정의한 인터페이스를 구현했다면, 래퍼 클래스 패턴(item18)을 사용하는 것이 좋다.
* 혹은 재정의 가능 메서드를 사용하지 않도록 변경하고, 문서화한다.
* 클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거해야할 때는, private '도우미 메서드'를 만들어 재정의 가능 메서드의 기능을 옮기고, 재정의 메서드 대신 도우미 메서드를 호출하도록 수정한다.

```java
public class Super {
    public Super() {
        //overrideMe();
        helper();
    }

    public void overrideMe() {
    	System.out.println("super method");
    }

    private void helper() {
    	System.out.println("super method");
    }
}
```
