# item 63) 문자열 연결은 느리니 주의하라

## '+' 연산 동작

* **String을 '+' 연산으로 이어붙이는 방법**은 아래와 같이 동작한다.

```java
// 두 문자열 b와 c를 더할 때
String a = b + c;

// 내부적으로는 아래와 같이 동작
String a = new StringBuilder(b).append(c).toString();
```

* for문을 돌 경우 수행 시간은 n^2 에 비례한다.
* String은 불변이기 때문에 두 문자열을 모두 복사해 새로운 객체로 연결하기 때문이다.

```java
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
        result += lineForItem(i);
    }
    return result;
}
```

## StringBuilder의 append 메서드

* **StringBuilder의 append 메서드를 직접 사용**하면 성능이 크게 개선된다.
* 이 방법의 수행 시간은 선형으로 비례한다.

```java
public String statement2() {
    StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        sb.append(lineForItem(i));
    }
    return sb.toString();
}
```

## 마무리

* 이 아이템의 요지는, for문으로 여러번 '+' 연산이 불릴 경우마다 새로운 객체가 생성되는 것을 방지하자는 것이다.
* 하지만, for문으로 돌리는 상황이 아니라면 '+' 연산을 사용해도 무방할 듯 하다.
