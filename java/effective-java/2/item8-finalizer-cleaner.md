# item8) finalizer와 cleaner 사용 자제

## finalizer, Cleaner

* finalizer는 GC를 관리해주는 스레드에서 호출한다.
* cleaner 역시 별도의 스레드에서 작업이 이루어진다.

## finalizer, cleaner의 단점

> finalizer와 cleaner는 예측할 수 없고 상황에 따라 위험할 수 있어 일반적으로 불필요하다.

### **예측 불가능**

* 얼마나 신속하게 finalizer와 cleaner가 수행될 지는 가비지 컬렉터의 알고리즘 구현에 따라 다르다.
* 수행 시점과 수행 여부를 보장하지 않아 상태를 영구적으로 수정하는 작업에서는 절대 사용하면 안된다.
  * ex) 데이터베이스의 영구 락 해제를 finalizer나 cleaner에 맡기면 안된다.
* finalizer의 경우 동작 중 발생한 예외는 무시되고 처리할 작업이 남아있어도 바로 종료되어 훼손된 객체 상태로 남을 수 있다.

### **finalizer 공격**

* finalize를 사용한 클래스에서는 공격으로 인한 보안 문제가 발생할 수 있다.
* 생성자나 직렬화 과정에서 예외가 발생해 훼손된 객체 내부에서 악의적인 하위 클래스의 finalizer가 수행될 수 있어 객체 생성을 막을 수 없다.
* final 클래스는 하위 클래스를 만들 수 없으므로 이 공격으로부턴 안전하다.
* final이 아닌 클래스는 final으로 선언된 finalize 메서드를 선언해두어 예방할 수 있다.

### **finalizer와 cleaner의 대안**

* AutoCloseable 인터페이스를 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하도록 한다.
* 일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 한다.
* 각 인스턴스는 자신이 닫혔는지를 추적하는 것이 좋다. close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 IllegalStateException을 던지는 것이다.

## finalizer와 cleaner 사용 방식

### **안전망 방식**

* 자원의 소유자가 close 메서드를 호출하지 않는 것에 대한 안전망 역할을 하도록 한다.
* FileInputStream, FileOutputStream, ThreadPoolExecutor 등이 안전망 역할의 finalizer를 제공한다.

### **네이티브 피어와 연결된 객체에서 사용**

> 네이티브 피어
>
> * 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
> * 자세한 설명은 [https://github.com/java-squid/effective-java/issues/8](https://github.com/java-squid/effective-java/issues/8) 참고

* 네이티브 피어는 자바 객체가 아니므로 가비지 컬렉터는 그 존재를 알지 못한다.
* 따라서 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못하므로 cleaner나 finalizer가 처리하기에 적합하다.
* 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당된다. 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드를 사용해야 한다.

### **AutoCloseable 클래스에서 cleaner를 안전망으로 활용**

* static으로 선언된 중첩 클래스가 Runnable을 구현하도록 하여, close 메서드나 cleanable.clean()이 중첩 클래스의 run 메서드를 호출하도록 한다.
* 가비지 컬렉터가 인스턴스를 회수할 때 클라이언트가 close 메서드 호출하지 않으면 Cleaner가 run 메서드를 호출할 수도 있다.
* 중첩 클래스의 인스턴스는 바깥 클래스의 인스턴스를 참조하면 순환 참조가 생겨 자동 청소될 수 없다. 중첩 클래스를 static으로 선언해 바깥 객체의 참조를 갖지 않도록 한다.
* try-with-resources 블록으로 클래스 생성을 하면 자동 청소가 필요하지 않다.
* cleaner를 통해 청소가 이뤄질지는 보장되지 않는다.
