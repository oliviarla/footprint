# item 35) ordinal 메서드 대신 인스턴스 필드를 사용하라

## ordinal 메서드

* 열거 타입에서 제공하는 해당 상수가 열거 타입에서 몇 번째 위치인지 인덱스를 반환하는 메서드
* ordinal()로 연결된 정수 값을 얻을 수 있다.
* 열거 타입 상수와 연결된 정숫값이 필요할 때에는 이 메서드보다는 인스턴스 필드에 직접 저장할 것
* 상수 선언 순서 변경이나 상수의 추가/제거에 따라 유동적으로 반영하기 어렵다.
* ordinal() 메서드를 사용하기보다는 아래 예제와 같이 직접 인스턴스 필드에 정수값을 지정해주도록 하자.

```java
public enum Ensembel{
    SOLO(1), DUET(2), TRIO(3);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size;}
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

## JPA Entity

* 이 부분을 읽으며 연관된 내용이라 책 내용은 아니지만 언급해본다.
* 기본적으로 JPA Entity 내부의 Enum 필드는 Ordinal 값을 데이터베이스에 저장하게 되어있다.
* 하지만 이 인덱스 값은 데이터베이스 상에서 알아보기 어렵기 때문에, 아래와 같이 Enumerated 어노테이션을 달면 Ordinal 대신 String으로 변환해 저장할 수 있다.
* 물론 이 부분은 요구사항에 따라 달라질 수 있기 때문에 선택하기 나름이다.

```java
@Entity
public class Student {

    @Enumerated(value = EnumType.STRING)
    private Grade grade;

}
```
