# item 75) 예외의 상세 메시지에 실패 관련 정보를 담으라

## 예외에 실패 메시지 담기

* 예외를 잡지 못하면, 자바 시스템은 예외의 스택 추적 정보를 자동으로 출력한다.
* toString 메서드를 호출해 얻는 문자열로, 예외 클래스 이름 뒤에 상세 메시지가 붙는다.
* 다음 예시는 아무런 메시지가 없는 예외이다.

```
Exception in thread "main" FoolException
    at Sample.sayNick(Sample.java:7)
    at Sample.main(Sample.java:14)
```

* 따라서 예외의 toString 메서드에 실패 원인에 관한 정보를 가능한 많이 담아 반환하는 것이 중요하다.
* 실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드 값을 실패 메시지에 담아야 한다.

> 당연하겠지만, 비밀번호나 암호키 같은 보안과 관련된 정보는 상세 메시지에 담아서는 안된다.

* 문서나 소스코드에서 얻을 수 있는 정보는 실패 메시지에 담을 필요 없다.
* 아래와 같이 예외의 생성자에서 실패를 쉽게 확인할 수 있도록 상세 메시지를 작성하면, 클래스 사용자는 직접 상세한 메시지를 만들지 않아도 된다.

```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index){
        //실패를 포착하는 상세 메시지 작성
        super("Lower bound: "+lowerBound +
        ", Upper bound: "+upperBound +
        ", Index: "+index);

        //프로그램에서 이용할 수 있도록 실패 정보 저장
        this.lowerBound = lowerBound;
        this.upperBound = upperBound;
        this.index = index;
}
```

* 예외 클래스에 저장해둔 실패 정보를 조회할 수 있는 접근자 메서드는 검사 예외에서 유용할 수 있다.
  * 비검사 예외는 대부분 로직의 오류로 인한 예외이므로, 비검사 예외의 상세 정보에 프로그램적으로 접근하는 경우는 드물다.
