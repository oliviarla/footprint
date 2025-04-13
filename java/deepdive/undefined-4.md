# 다이나믹 프록시

## JDK 다이나믹 프록시

* 런타임에 특정 인터페이스를 구현하는 클래스 또는 인스턴스를 만드는 기술이다.
* 아래와 같이 `Proxy.newProxyInstance` 메서드를 이용해 동적으로 프록시 클래스를 만들 수 있다.

```java
BookService bookService = (BookService) Proxy.newProxyInstance(BookService.class.getClassLoader(), new Class[]{BookService.class},
    new InvocationHandler() {
        BookService bookService = new DefaultBookService();
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.getName().equals("rent")) { 
            System.out.println("before method call");
            Object invoke = method.invoke(bookService, args);
            System.out.println("after method call");
            return invoke;
        }
        return method.invoke(bookService, args); 
        }
    });
```

## 클래스 기반 다이나믹 프록시

* 상속을 사용하지 못하는 경우 프록시를 만들 수 없다.
* 인터페이스가 있을 때는 인터페이스의 프록시를 만들어 사용해야 한다.

### CGLIB

* MethodInterceptor로 핸들러를 만들어 어떤 작업을 수행할 지 Enhancer에 넘기면 클래스의 프록시 객체를 얻을 수 있다.

```java
MethodInterceptor handler = new MethodInterceptor() {
    BookService bookService = new BookService();
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("before method call");
        return method.invoke(bookService, objects);
    } 
};
BookService bookService = (BookService) Enhancer.create(BookService.class, handler);
```

### ByteBuddy

* 바이트버디를 이용해서도 다이나믹 프록시를 생성할 수 있다.

```java
Class<? extends BookService> proxyClass = new ByteBuddy().subclass(BookService.class)
    .method(named("rent"))
    .intercept(InvocationHandlerAdapter.of(new InvocationHandler() {
        BookService bookService = new BookService();
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("before method call");
            return method.invoke(bookService, args);
        }
    }))
    .make()
    .load(BookService.class.getClassLoader())
    .getLoaded();
BookService bookService = proxyClass.getConstructor(null).newInstance();
```
