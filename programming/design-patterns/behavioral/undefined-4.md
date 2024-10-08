# 비지터 패턴

## 접근

* 다양한 객체에 새로운 기능을 추가해야 할 때 모든 객체에 추가하는 대신 비지터 객체에만 새로운 기능을 추가한다.
* 각각의 객체에 기능을 일일이 추가하는 것은 수정 과정에서 오류가 발생할 가능성도 있고 단일 책임 원칙을 깨게 될 수도 있다.

## 개념

* 기존 클래스 대신 비지터 클래스에 새로운 행동을 추가하고, 비지터 클래스 메서드의 인자로 기존 클래스 객체를 입력받아 필요한 데이터를 사용할 수 있다.
* 이 때 객체로부터 원하는 데이터를 받아와야만 하기 때문에 캡슐화가 깨질 수 있다.
* 더블 디스패치라는 방법을 사용하여 타입 확인하는 조건문 없이 클래스에 맞는 비지터 메서드를 호출하도록 한다. 비지터 객체를 인자로 받는 accept 메서드를 각 클래스에 구현하고, 내부에서는 알맞는 비지터 메서드를 호출하면 된다.

```java
public class City {
    public void accept(Visitor v) {
        v.doForCity(this);
    }
    // ...
}

public class Industry {
    public void accept(Visitor v) {
        v.doForIndustry(this);
    }
    // ...
}
```

* 아래는 비지터 패턴의 구성요소이다.
  * 비지터 인터페이스는 여러 클래스들을 인자로 받는 비지터 메서드들을 선언한다.
  * 비지터 구현체는 다양한 클래스들에 대한 동일한 행동을 할 수 있도록 메서드를 구현한다.
  * 각 클래스들은 accept 메서드를 가진 Element 인터페이스를 구현하여 알맞는 비지터 메서드가 호출되도록 한다.
  * 클라이언트는 일반적으로 컬렉션 혹은 컴포지트 트리 등을 통해 비지터를 적용하게 된다.

<figure><img src="../../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

## 장단점

* 장점
  * 구조를 변경하지 않으면서 복합 객체 구조에 새로운 기능을 추가할 수 있다.
  * 새로운 기능을 손쉽게 추가 가능하다.
  * 비지터가 수행하는 기능과 관련된 코드를 한 곳에 모을 수 있다.
* 단점
  * 복합 클래스의 캡슐화가 깨진다.
  * 컬렉션 내 모든 항목에 접근하는 트레버서로 인해 복합 구조를 변경하기 어려워진다.

## 예시

* 자바의 바이트 코드를 조작하는 라이브러리인 ASM에서는 ClassReader라는 클래스를 통해 class 파일 내용을 읽고 파싱한 후 ClassNode라는 비지터 구현체의 visit 메서드들을 호출하게 된다. ClassNode는 class 파일으로부터 읽어들인 정보들을 읽기 쉽게 저장하기 때문에 `visit`, `visitOuterClass`, `visitAnnotation` 등의 메서드를 통해 ClassReader로부터 전달된 정보들을 가공하여 필드에 저장하게 된다.

```java
ClassReader classReader = new ClassReader("MyClass");
ClassNode classNode = new ClassNode();
classReader.accept(classNode, 0); // classNode에 읽어온 정보를 저장한다!
```
