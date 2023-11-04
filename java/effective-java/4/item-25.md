# item 25) 톱레벨 클래스는 한 파일에 하나만 담으라

* 톱레벨 클래스: 이너 클래스가 아닌, 말그대로 가장 최상단에 존재하는 클래스이다.
* 하나의 소스파일에 여러개의 톱레벨 클래스를 선언하더라도 컴파일 오류는 발생하지 않는다.
* 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문에 톱레벨 클래스들을 서로 다른 소스 파일로 분리해야 한다.

```java
class Utensil {
    static final String NAME = "pan";
}
class Dessert {
    static final String NAME = "cake";
}
```

* 여러 톱레벨 클래스를 한 파일에 담고 싶다면 다음과 같이 정적 멤버 클래스 사용을 고려할 것

```java
public class Test {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
	private static class Utensil {
		static final String NAME = "pan";
	}
	private static class Dessert {
		static final String NAME = "cake";
	}
}
```
