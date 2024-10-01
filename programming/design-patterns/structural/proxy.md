# 프록시 패턴

## 접근

## 개념

* 특정 객체로의 **접근을 제어**하는 대리인 역할을 하는 객체를 제공하는 디자인 패턴이다.
* 클라이언트가 실제 객체의 메서드를 호출했을 때 프록시 객체가 해당 호출 중간에 가로채 필요한 로직을 수행할 수 있다.
* 생성하기 어려운 객체나 보안이 중요한 객체 등에 대해 접근을 제어하는 대리인 객체를 만들어 사용할 수 있다.
* 원격(remote) 프록시를 사용해 원격 객체로의 접근을 제어할 수 있다. Java의 RMI가 이 방식에 해당된다.
* 가상(virtual) 프록시를 사용해 생성하기 힘든 자원으로의 접근을 제어할 수 있다.
  * 생성하는 데에 비용이 많이 드는 객체를 미리 만들어두지 않고 정말 필요할 때 프록시 내부에서 생성하도록 한다.
* 보호(protection) 프록시를 사용해 접근 권한이 필요한 자원으로의 접근을 제어할 수 있다.
* 이외에도 방화벽 프록시, 캐싱 프록시, 동기화 프록시 등 다양한 프록시 방법이 존재한다.
* 실제 클래스와 프록시 클래스는 같은 인터페이스를 구현하여 실제 객체가 들어갈 자리에 프록시 객체를 넣을 수 있도록 한다.
* 프록시 클래스에는 실제 객체의 참조를 가지며 필요할 때 해당 객체를 사용할 수 있다.

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

* 언뜻 보면 특정 클래스를 감싸고 해당 클래스의 인터페이스를 구현한다는 점에서 데코레이터와 비슷할 수 있지만 데코레이터는 클래스에 새로운 행동을 추가하는 용도로 쓰이고 프록시는 클래스의 접근을 제어하는 용도로 쓰인다는 점에서 차이가 있다.
* 보호 프록시는 클라이언트 역할에 따라 객체에 있는 특정 메서드로의 접근을 제어한다. 따라서 클라이언트에게 인터페이스의 일부분만 제공하는 형태이므로 어댑터 패턴과 유사하다.

## 장단점

* 장점
  * 클라이언트는 실제 객체를 사용하는지 프록시 객체를 사용하는지 신경쓰지 않아도 된다.
  * 실제 객체가 준비되지 않은 상태에서도 객체를 사용할 수 있는 것 처럼 다뤄질 수 있다.
* 단점
  * 새로운 클래스를 도입해야 하므로 코드가 복잡해질 수 있다.
  * 부가적인 코드(ex. 초기화가 오래걸리는 객체 생성 등)에 의해 서비스의 응답이 느려질 수 있다.

## 사용 방법

### 동적 프록시 사용하기

* 자바에서는 동적 프록시 기능을 제공한다. 프록시의 메서드가 호출되었을 때 할 일을 지정해주는 핸들러를 만들면 프록시 클래스가 실행 중에 생성되도록 할 수 있다.
* InvocationHandler 인터페이스를 구현한 클래스에 invoke 메서드를 작성하고, 자바에서 제공되는 newProxyInstance 메서드를 통해 Proxy 객체를 생성해 사용할 수 있다. Proxy 객체는 메서드 호출을 받으면 항상 InvocationHandler에 작업을 위임하게 된다.

```java
@RequiredArgsConstructor
public class OwnerInvocationHandler implements InvocationHandler {
    private final Person person;
    
    public Object invoke(Object proxy, Method method, Object[] args) throws IllegalAccessException {
        try {
            if (method.getName().startsWith("get")) {
                return method.invoke(person, args);
            } else if (method.getName().startsWith("setRating")) {
                throw new IllegalAccessException();
            } else if (method.getName().startsWith("set")) {
                return method.invoke(person, args);
            }
        } catch (InvocationTargetException e) { // 메서드 수행 중 에러 났을 때
            e.printStackTrace();
        }
        return null;
    } 
}
```

```java
Person person = new PersonImpl("kim", 32);

Person proxyPerson = (Person) Proxy.newProxyInstance(person.getClass().getClassLoader(),
                                                     person.getClass().getInterfaces(),
                                                     new OwnerInvocationHandler(person));
```

## 예시





