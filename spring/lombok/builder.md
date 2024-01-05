# @Builder

## 개념

* 클래스 레벨이나 생성자 레벨에 붙여주면 파라미터를 활용해 빌더 패턴을 자동으로 구현해주는 어노테이션
* 클래스 생성 시 많은 필드를 입력해야 하는 상황에서 가독성이 좋아 유용하다.
* private 생성자를 가지는 `<ClassName>Builder` 라는 이름의 내부 빌더 클래스를 생성하여 빌더 패턴을 구현해준다.

```java
    ...
    public static class StudentBuilder {

        private String name;
        private int age;
    
        StudentBuilder() {
        }
    
        public StudentBuilder name(String name) {
            this.name = name;
            return this;
        }
    
        public StudentBuilder age(int age) {
            this.age = age;
            return this;
        }
    
        public Student build() {
            return new Student(name, age);
        }
    
        public String toString() {
            return "Student.StudentBuilder(name=" + this.name + ", age=" + this.age + ")";
        }
    }
    ...
```

## 특징

* 빌더 클래스의 내부 메서드 생성 기준은 어노테이션을 적용하는 레벨에 따라 달라진다.
  * **클래스 레벨**에서 선언 시 가능한 모든 필드에 대하여 빌더 메서드를 생성한다.
  * **생성자 레벨**에서 선언 시 생성자의 파라미터 필드에 대해서만 빌더 메서드를 생성한다.
* 빌더 클래스 내부의 필드들은 final이 아닌 지역 변수이다. 왜냐하면 한 번 설정한 속성을 여러 번 메서드를 호출하여 다시 설정할 수 있어야 하기 때문이다.
* 빌더 메서드로 초기화하지 않은 필드는 생성자로 객체를 생성할 때 전역 변수의 기본값인 null이 저장되므로 주의해야 한다.

## @Builder.Default

* 반드시 초기화되어야 하는 필드의 경우, `@Builder.Default` 속성을 사용하거나 선언 시점 또는 생성자에서 초기화해주어야 한다.

```java
@Builder
public class Student {

    private String name;
    private int age;
    @Builder.Default
    private String teacher = "Mrs. White";

        // ...
}
```

## GOF 빌더 패턴과 다른 점

* Lombok이 생성해주는 빌더 클래스는 사실 GOF 빌더 패턴의 모양과 다르다.
* GOF 빌더 패턴의 경우 아래와 같은 클래스들이 사용된다.
  * Builder : Product 객체 생성을 위한 정의가 담긴 인터페이스
  * ConcreteBuilder : Builder 인터페이스의 구현체
  * Director : Builder를 사용해 객체를 생성하는 클래스
  * Product : Director가 Builder로 만들어낸 최종 결과물 클래스

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### 출처

{% embed url="http://www.javabyexamples.com/delombok-builder" %}

{% embed url="https://johngrib.github.io/wiki/pattern/builder/#gof-%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4%EC%9D%98-%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4" %}
