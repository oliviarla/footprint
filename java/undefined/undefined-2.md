# 바이트 코드 조작

## 바이트 코드 조작

### 라이브러리

* ASM
  * visitor 패턴, adapter 패턴을 이해해야 사용하기 쉽다.
* Javaassist
* ByteBuddy
  * 가장 간단하게 바이트 코드를 조작할 수 있다.

### 조작 예제

* 아래와 같이 빈 문자열을 반환하는 메서드를 가진 Moja 클래스의 바이트코드를 조작하여 Rabbit을 반환하도록 만들 수 있다.

```java
public class Moja {

    public String pullOut() {
        return "";
    }
}

```

```java
try {
    new ByteBuddy.redefine(Moja.class)
        .method(named("pullOut")).intercept(FixedValue.value("Rabbit"))
        .make().saveIn(new File("/User/workspace/example/target/classes/"));
} catch (IOException e) {
    e.printStackTrace();
}
```

## 자바 에이전트

