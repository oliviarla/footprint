# 리플렉션

## 클래스 정보 조회

* `Class<T>`라는 타입을 통해 클래스의 모든 정보(ex. 필드, 메서드, 상위 클래스, 인터페이스, 어노테이션, 생성자 등)를 조회할 수 있다.
* 클래스를 문자열을 통해 읽을 수 있으며, `Class.forName("MyClass")` 와 같이 사용할 수 있다. 만약 해당 클래스가 없다면 ClassNotFoundException이 발생한다.

## 어노테이션과 리플렉션

* `@Retention` : 어노테이션을 유지하는 기간을 설정한다.
  * SOURCE, CLASS, RUNTIME 속성을 사용 가능하며 기본적으로 RUNTIME 속성이 사용된다.
  * SOURCE 레벨은 말그대로 소스 코드 까지 어노테이션을 유지하는 것이다.
  * CLASS 레벨은 바이트 코드까지 어노테이션을 유지하는 것이다.
  * RUNTIME 레벨은 실제 런타임까지 어노테이션을 유지하는 것으로, 리플렉션으로 어노테이션을 읽으려면 이 속성을 사용해야 한다.
* `@Inherited` : 부모 클래스에 어노테이션을 붙였을 때 자식 클래스에도 어노테이션을 적용한다.
* `@Target` : 어노테이션을 어디에서까지 사용할 수 있는지 설정한다.
  * TYPE, FIELD, METHOD 등 어느 곳에 어노테이션을 붙일 수 있는지 지정한다.
* 어노테이션은 primitive 타입 혹은 boxing된 타입을 필드로 가질 수 있다.

```java
public @interface PersonAnnotation {
    String name() default "kim";
    int age() default 20;
}
```

* 리플렉션을 통해 어노테이션의 정보를 얻을 수 있다.

```java
PersonAnnotation personAnnotation = Son.class.getAnnotations(); // 상속받은 어노테이션까지 조회
String nameOfSon = personAnnotation.name();
int ageOfSon = personAnnotation.age();

Mom.class.getDeclaredAnnotations(); // 자신 클래스에 붙은 어노테이션만 조회
```

## 클래스 정보 수정 및 실행

### 객체 생성하기

```java
Constructor<NullValue> constructor = NullValue.class.getDeclaredConstructor();
constructor.setAccessible(true); // private일 경우 접근 가능하도록 허용해주어야 한다.
NullValue nullValue = constructor.newInstance();
```

### 객체의 필드 조회/수정하기

```java
Book book = (Book) Book.class.getConstructor(String.class).newInstance("myBook");
Field name = Book.class.getDeclaredField("name");
name.setAccessible(true);
String nameOfBook = name.get(book); // 필드 조회하기
name.set(book, "myUpdatedBook"); // 필드 수정하기
```

### 메서드 실행하기

```java
Book book = new Book();
Method setTotalPages = book.getClass().getDeclaredMethod("setTotalPages", long.class);
setTotalPages.setAccessible(true); // private일 경우 접근 가능하도록 허용해주어야 한다.
setTotalPages.invoke(book, 150);
```

## 주의점

* 지나친 사용은 성능 이슈를 야기할 수 있으니 반드시 필요한 경우에만 사용해야 한다.
* 컴파일 타임에 확인되지 않은 문제가 런타임에 발생할 수 있다.
* 접근 지시자를 무시할 수 있다.
