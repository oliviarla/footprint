# item 88) readObject 메서드는 방어적으로 작성하라

* readObject 메서드는 실질적으로 또다른 public 생성자이므로 다른 생성자만큼 주의깊게 다뤄야 한다.
* 인수가 유효한지 검사하고, 매개변수를 반드시 방어적 복사하여 (item 50) 잘못된 역직렬화를 막을 수 있다.

#### 가변 공격

* 정상 객체 인스턴스에서 시작된 바이트 스트림의 끝에 private 필드로의 참조를 추가하면 가변 객체 인스턴스를 만들어낼 수 있다.
* 아래는 직렬화된 바이트스트림에 악의적인 객체 참조를 추가해 내부 값에 접근 가능한 예제이다.

```arduino
public class MutablePeriod {
//Period 인스턴스
public final Period period;

public final Date start;
public final Date end;

public MutablePeriod() {
	try {
        ByteArrayOutputStream bos =new ByteArrayOutputStream();
        ObjectOutputStream out =new ObjectOutputStream(bos);

				// 1) Period 인스턴스 직렬화
        out.writeObject(new Period(new Date(),new Date()));

				// 2) start, end 필드에 대한 참조 추가
        byte[] ref = {0x71, 0, 0x7e, 0, 5};
        bos.write(ref);
        ref[4] = 4;
        bos.write(ref);
				
				// 3) Period 역직렬화 후 Date 참조를 훔친다.
        ObjectInputStream in =new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
        period = (Period)in.readObject();
        start = (Date)in.readObject(); //Period 내부의 start 데이터 탈취
        end = (Date)in.readObject(); //Period 내부의 end 데이터 탈취
    } catch (IOException | ClassNotFoundException e) {
				throw new AssertionError(e);
    }
}
```

*   불변식과 불변 성질을 지키기 위해 모든 private 가변 요소를 방어적으로 복사하면 해결할 수 있다.

    * final 필드의 경우 final 한정자를 제거해야 방어적 복사가 가능하다.

    ```java
    private void readObject(ObjectInputStream s)throws IOException, ClassNotFoundException {
      s.defaultReadObject();

    	// 가변 요소들을 방어적으로 복사한다.
      start =new Date(start.getTime());
      end =new Date(end.getTime());

    	// 불변식을 만족하는지 검사한다.
    	if (start.compareto(end) > 0) {
    		throw new InvalidObjectException(start + " after " + end);
    	}
    }
    ```

#### 기본 readObject 메서드

* transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮다면 사용 가능
* final이 아닌 직렬화 가능한 클래스라면 readObject 메서드도 재정의 가능한 메서드를 호출해서는 안 된다. (item 19)
  * 하위 클래스의 상태가 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행될 수 있고, 이는 프로그램의 오작동으로 이어질 수 있다.
* 기본 readObject 메서드를 사용할 수 없다면, **직렬화 프록시 패턴을 사용하거나 커스텀 readObject** 메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행해야 한다.
