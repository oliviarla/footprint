# item 85) 자바 직렬화의 대안을 찾으라

### 직렬화

#### 개념

* 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)을 아우르는 것
* 시스템적으로 보면, JVM의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 바이트 형태로 변환하는 기술과, 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 이야기한다.

#### 사용하는 이유

* 자바 시스템 개발에 최적화되어있어 복잡한 데이터 구조의 클래스의 객체라도 직렬화 기본 조건만 지키면 큰 작업 없이 바로 직렬화/역직렬화 가능하다.
* 데이터 타입이 자동으로 맞춰지기 때문에 이 부분에 대해 신경을 쓰지 않아도 된다.
* 프레임워크를 사용하지 않는 경우에는 CSV, JSON을 사용하는 것보다 편할 것 같다.

### 직렬화의 문제점

* **공격 범위가 너무 넓고, 지속적으로 더 넓어지기 때문에 방어하기 어렵다**
* readObject() 메서드는 Serializable 인터페이스를 구현한 클래스패스 안에 거의 모든 타입의 객체를 만들어 낼 수 있다. 바이트 스트림을 역직렬화하는 과정에서 이 메서드는 그 타입들 안의 모든 코드를 수행할 수 있고, 이로 인해 그 타입들의 **코드 전체가 공격 범위**에 들어가게 된다. (자바 표준 라이브러리와 서드 파티 라이브러리 등 모두 포함)
*   여러 가젯 메서드를 사용해 가젯 체인을 구성할 수 있으며, 강력한 가젯 체인은 시스템을 마비시키고 그로 인해 막대한 피해를 볼 수 있다.

    > 가젯 메서드: 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드

#### 역직렬화 폭탄

* 역직렬화에 시간이 오래 걸리는 짧은 스트림을 역직렬화하는 스트림
* 아래 예시는 바이트 스트림으로 직렬화하는데는 시간이 별로 걸리지 않지만, 역직렬화시 hashCode 메서드를 2^100번 넘게 호출해야 하므로 영원히 계속된다.
* 여러 객체를 생성한다면 스택 깊이 제한에 걸려 프로그램에 장애가 발생할 것이다.

```java
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();

    for (int i=0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();

        t1.add("f"); // t1: {"f"}, t2: {}
        s1.add(t1); s1.add(t2); // s1:{{"f"}, {}}, root:{{"f"}, {}}

        s2.add(t1); s2.add(t2); // s2:{{"f"}, {}}
        s1 = t1; s2 = t2; // s1:{"f"}, s2: {}, root:{{"f"}, {}}
    }
    return serialize(root);
}
```

### 해결 방안

* 새로운 시스템에는 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘이 많이 있기 때문에 **직렬화 사용을 자제하는 것이 좋다.**
* 신뢰할 수 없는 데이터는 절대 역직렬화하면 안된다. (역직렬화 폭탄)

#### 크로스-플랫폼 구조화된 데이터 표현

cross-platform structured-data representation

* 자바 직렬화와 다른 메커니즘을 가진 객체와 바이트 시퀀스 변환 시스템
* 자바 직렬화의 위험성이 없으며 훨씬 간단하다.
* 다양한 플랫폼 지원, 우수한 성능, 풍부한 지원 도구 등을 제공
* 속성-값 쌍의 집합으로 구성된 간단하고 구조화된 데이터 객체를 사용
* JSON
  * 텍스트 기반 표현에 효과적이고 사람이 읽을 수 있다
* 프로토콜 버퍼
  * 문서를 위한 스키마를 제공하며 효율이 좋다

#### 역직렬화 필터링

* 데이터 스트림이 역직렬화되기 전에 필터를 설치하는 기능
* 블랙리스트에 기록된 클래스를 거부하거나, 화이트리스트에 기록된 클래스만 수용한다.
* 이미 알려진 위험으로부터만 보호할 수 있는 블랙리스트보다 화이트리스트를 사용하는 것이 좋다.
* 메모리를 과하게 사용하거나 객체 그래프가 너무 깊어지는 상황으로부터 보호한다.
* 역직렬화 폭탄은 걸러내지 못한다.

출처

* [https://techblog.woowahan.com/2550/](https://techblog.woowahan.com/2550/)

## item 86) Serializable을 구현할지는 신중히 결정하라

### Serializable 구현의 문제점

#### 1) 릴리스 후 수정 어려움

* Serializable을 구현하면 릴리스 한 뒤에는 수정하기 어려워진다.
* Serializable을 구현한 클래스는 하나의 공개 API가 되므로 구현한 직렬화 형태도 영원히 지원해야 한다.
* **직렬화 가능 클래스를 만든다면 고품질의 직렬화 형태도 함께 설계해야 한다.**
* 자바의 기본 직렬화 방식을 사용하면 직렬화 형태는 적용 당시 클래스의 내부 구현 방식에 종속된다.
* 원래의 직렬화 형태를 유지하면서 내부 구현을 수정할 수도 있지만, 이는 어렵고 소스코드도 지저분해진다.
* 클래스의 private와 package-private 인스턴스 필드마저 API로 공개되어 캡슐화가 깨진다.
* 나중에 내부 구현을 수정하면 직렬화 형태가 달라져 직렬화-역직렬화에 실패할 수 있다.
* ex) serialVersionUID 자동 생성 이슈
  * Serializable 클래스의 버전을 기억해 클래스와 직렬화 된 객체가 호환되는지 확인하는 식별자
  * 직접 번호를 명시하지 않는 경우 런타임 시 자동으로 생성된다.
  * 자동 생성될 때 클래스의 이름, 구현한 인터페이스 등이 고려되기 때문에 이들 중 하나라도 변경되면 UID 값도 달라진다. 따라서 쉽게 호환성이 깨지고 런타임에 InvalidClassException이 발생한다.

#### 2) 버그와 보안 취약점 발생 위험

* 기본 역직렬화는 전면에 드러나지 않는 **숨은 생성자**이기 때문에 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다.

#### 3) 신버전 릴리스 시 테스트 요소 증가

* 직렬화 가능 클래스가 수정되면 신버전/구버전 인스턴스 간의 양방향 직렬화/역직렬화가 가능한지 검사해야 한다.
* 테스트해야 할 양이 직렬화 가능 클래스 수와 릴리스 횟수에 비례해 증가한다.

#### 4) 구현 여부는 쉽게 결정할 사안이 아니다.

* Serializable을 반드시 구현해야 하는 클래스는 구현에 따르는 이득과 비용을 잘 고려해 설계해야 한다.
*   BigInteger와 Instant 같은 **'값'** 클래스와 컬렉션 클래스들은 Serializable을 구현하고, 스레드 풀처럼 \*\*'동작'\*\*하는 객체를 표현하는 클래스는 대부분 Serializable을 구현하지 않았다.

    💡 값을 나타내는 클래스는 아무래도 Spring의 entity와 비슷하게 외부와 통신하는 경우가 많을 것 같다.

#### 5) 상속용으로 설계된 클래스/인터페이스는 Serializable 확장 불가

* 해당 클래스/인터페이스를 구현하는 대상에게 Serializable의 문제점들을 그대로 전이하기 때문
* 하지만 Serializable을 구현한 클래스만 지원하는 프레임워크를 사용하는 상황이라면 이 규칙을 지키지 못하는 경우도 생긴다.
*   ex) Throwable

    * Throwable은 서버가 RMI를 통해 클라이언트로 예외를 보내기 위해 Serializable을 구현했다.

    > **RMI**(Remote Method Invocation) 자바와 개발환경을 사용하여 서로 다른 컴퓨터들 상에 있는 객체들이 분산 네트워크 내에서 상호 작용하는 객체지향형 프로그램을 작성할 수 있는 방식이다.
* 직렬화, 확장(extends)이 모두 가능한 클래스에서 불변식을 보장해야 한다면 finalize 메서드를 하위 클래스가 재정의하지 못하도록 자신이 재정의하면서 final로 선언해야 finalizer 공격을 피할 수 있다.
*   필드 중 원시 값으로 초기화되면 위배되는 불변식이 있다면 readObjectNoData를 반드시 추가해야 한다.

    ```java
    private void readObjectNoData() throws InvalidObjectException {
    	throw new InvalidObjectException("스트림 데이터가 필요합니다.");
    }
    ```

### Serializable을 구현하지 않는 상위 클래스

* 상속용 클래스에서 직렬화를 지원하지 않지만, 하위 클래스에서 직렬화를 지원하려 한다면 부담이 늘어난다.
* 이런 클래스를 역직렬화하려면 상위 클래스의 매개변수가 없는 생성자를 제공하거나, 하위 클래스에서 직렬화 프록시 패턴을 사용해야 하기 때문이다.

### 내부 클래스와 직렬화

* 내부 클래스에는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 **자동으로 추가**되어 기본 직렬화 형태가 유동적이다.
* 정적 멤버 클래스는 컴파일러에 의해 자동 추가되는 필드가 따로 없으므로 Serializable을 구현해도 괜찮다.

## item 87) 커스텀 직렬화 형태를 고려해보라

### 기본 직렬화 vs 커스텀 직렬화

* 기본 직렬화 형태는 어떤 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 그 객체들이 연결된 위상까지 기술한다.
* 이상적인 직렬화 형태라면 물리적인 모습이 아닌 **논리적인 모습만을 표현**해야 한다.
* 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태를 사용해도 되지만, 다르다면 커스텀 직렬화 형태로 지정해주어야 한다.
* 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.

### 기본 직렬화 형태 사용하는 경우

* 직접 설계해도 기본 직렬화 형태와 거의 같은 결과가 나올 때
*   ex) Name 클래스는 이름, 성, 중간 이름이라는 3개의 문자열로 구성된 **논리적 구성요소**로 표현되며, 위 코드는 이 논리적 구성요소를 물리적으로 정확히 반영했다.

    ```java
    public class NameimplementsSerializable {

    /**
         * 성. null이 아니어야함
         *@serial
         */private final String lastName;

    /**
         * 이름. null이 아니어야 함.
         *@serial
         */private final String firstName;

    /**
         * 중간이름. 중간이름이 없다면 null.
         *@serial
         */private final String middleName;
    }
    ```

    > **@serial 태그**로 기술한 javadoc 내용은 API 문서에서 직렬화 형태를 설명하는 특별한 페이지에 기록됨
* **불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.**
  * 메서드 내에서 null이 아님을 보장하도록 할 수 있다.

### **물리적 표현과 논리적 표현의 차이**

* **객체의 물리적 표현과 논리적 표현의 차이가 큰 객체에 기본 직렬화 형태를 적용하면 문제가 발생한다.**
* 아래 예제 클래스는 논리적으로 문자열을 표현하고, 물리적으로는 문자열을 이중 연결 리스트형태로 연결한다.

```java
public final class StringList implements Serializable {
	private int size = 0;
	private Entry head =null;
	
	private static class Entry implements Serializable {
	        String data;
	        Entry next;
	        Entry previous;
	    }
}
```

#### 공개 API가 현재 내부 표현 방식에 얽매임

* 기본 직렬화 형태를 사용하면 private 클래스인 StringList.Entry가 공개 API가 돼버린다.
* 추후 내부 표현방식을 바꾸더라도 StringList의 클래스는 여전히 연결 리스트로 표현된 입력을 처리할 수 있어야 한다.

#### 너무 많은 공간 차지

* 위 코드의 직렬화 형태는 연결 리스트의 모든 Entry와 연결 정보를 기록하지만 Entry와 연결 정보는 내부 구현에 속하니 직렬화 형태에 포함할 가치가 전혀 없다.
* 기본 직렬화 사용 시 이러한 정보들이 포함되기 때문에 디스크 저장 속도, 네트워크 전송 속도가 느려진다.

#### 과도한 시간 소요

* 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없어 그래프를 직접 순회해봐야 하므로 객체의 형태에 따라 순회 시간이 너무 많이 걸릴 수도 있다.

#### 스택 오버플로 발생

* 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 과정에서 스택 오버플로를 일으킬 수 있다.
* 실행할 때마다 스택 오버플로가 발생하는 리스트의 최소 크기가 달라질 수 있다. 즉, 플랫폼에 따라 발생여부가 달라진다.

### 커스텀 직렬화

* 단순히 리스트가 포함한 문자열의 개수를 적은 다음, 그 뒤로 문자열들을 나열하는 **논리적 모습으로 커스텀 직렬화**하면 문제 발생을 막을 수 있다.
* writeObject와 readObject 메서드를 통해 직렬화 형태를 처리한다.
* 필드에 transient 한정자를 붙여 기본 직렬화 형태에 포함되지 않도록 한다.
* 커스텀 직렬화 구현 전에 **기본 직렬화를 수행하는 defaultWriteObject와 defaultReadObject 메서드를 먼저 호출**해, 향후 transient가 아닌 인스턴스 필드가 추가되더라도 호환 가능하게 한다.

```java
public final class StringList implements Serializable {
	private transient int size = 0;
	private transient Entry head =null;
	
	private static class Entry {
	        String data;
	        Entry next;
	        Entry previous;
	    }
	
	// Entry에 저장할 때 사용
	public final void add(String s) {...}

	private void writeObject(ObjectOutputStream s) throws IOException {
      s.defaultWriteObject();
			// 리스트 크기 기록
      s.writeInt(size);

			// 커스텀 직렬화: 모든 Entry 원소를 순서대로 write
			for (Entry e = head; e !=null; e = e.next)
          s.writeObject(e.data);
  }
		
	private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
      s.defaultReadObject();
			int numElements = s.readInt();

			// 커스텀 역직렬화: 모든 원소를 순서대로 read해 Entry에 저장
			for(int i = 0; i < numElements; i++) {
          add((String) s.readObject());
      }
  }
}
```

* 기본 직렬화 방식 공간의 절반을 차지하며, 수행 속도도 빨라진다. 또 스택 오버플로가 발생하지 않기 때문에 직렬화의 크기 제한이 없어진다.

### 커스텀 직렬화 시 주의 사항

#### 불변식이 깨지는 객체

* 예를 들어 해시 테이블은 key-value 엔트리를 담은 해시 버킷을 차례로 나열한 형태로 구성된다. 버킷에 어떤 엔트리를 담을지는 해시 코드가 결정하는데, 이러한 해시 코드는 구현 방식에 따라 달라질 수 있다.
* 세부 구현에 따라 불변식이 깨지는 객체는 정확성을 깨트린다. 이런 객체를 기본 직렬화한 후 역직렬화하면 불변식이 심각하게 훼손된 객체들이 생길 수 있다.

#### transient 한정자

* transient를 선언해도 되는 필드에는 모두 transient 한정자를 붙여 defaultWriteObject 메서드를 호출 시 사용되지 않도록 한다.
* JVM을 실행할 때마다 값이 달라지는 네이티브 자료구조를 가지는 필드(long 필드)나 캐시 된 해시 값과 같은 다른 필드에서 유도되는 필드 등도 transient를 붙여야 한다.
* 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 tranisent 한정자를 생략해야 한다.
  * 논리적 상태는 커스텀 직렬화 로직에 의해 read/write되지만 물리적 상태는 그렇지 않다.

#### transient 필드 초기화값

* 초기화 기본값(null, 0, false 등)을 그대로 사용하면 안되는 경우 readObject 메서드에서 defaultReadObject를 호출한 다음, 해당 필드를 원하는 값으로 복원해야 한다.
* 지연 초기화를 통해 값을 사용할 때 초기화할 수도 있다.

#### 동기화 메커니즘

* 기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.
* 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하려면 **writeObject도 synchronized로 선언**해야 한다.
* writeObject 메서드 안에서 동기화하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라하지 않으면 교착상태가 발생할 수 있다.

#### 직렬화 가능 클래스에 직렬 버전 UID 명시하기

* 직렬 버전 UID를 명시하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제(item 86)를 해결할 수 있다.
* 런타임에 이 값을 생성하는 시간을 단축시켜 성능이 빨라진다.

```java
public class Foo implements Serializable {
	private static final long serialVersionUID = 5130240674615883456L;
}
```

* 구버전과 호환성을 유지하고 싶다면, 구버전에서 생성된 UID 값을 그대로 저장해야 한다.
* 기본 버전 클래스와 호환성을 끊고 싶다면 단순히 UID 값을 바꿔주면 된다.
  * 기존 버전 직렬화 인스턴스 역직렬화할 때 InvalidClassException 발생
* 일반적인 상황이라면 직렬 버전 UID를 절대 수정하지 말 것

## item 88) readObject 메서드는 방어적으로 작성하라

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

## item 89) 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

* `implements Serializable` 을 추가하면 무조건 싱글톤이 아니게 된다.
* 어떤 readObject 메서드를 사용해도 클래스가 초기화될 때의 인스턴스와 다른 인스턴스가 반환된다.

### readResolve

* 역직렬화한 객체의 클래스가 적절히 정의해두었다면, 새로 생성된 객체를 인수로 이 메서드가 호출되어, 새로 생성된 객체를 반환하지 않고 readResolve로부터 얻은 객체 참조를 반환한다.
*   인스턴스 통제 목적으로 이 메서드를 사용한다면, 모든 필드가 transient로 선언되야 한다.

    * item 88의 가변 공격 방식과 비슷하게 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 훔쳐올 수 있다.

    💡 자세한 과정

    1. readResolve 메서드와 인스턴스 필드 하나를 포함한 **도둑 클래스**를 만든다.
    2. 도둑 클래스의 인스턴스 필드는 직렬화된 싱글턴을 참조하는 역할을 한다.
    3. 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 도둑의 인스턴스 필드로 교체한다.
    4. 싱글턴이 도둑을 포함하므로 역직렬화시 도둑 클래스의 readResolve가 먼저 호출된다.
    5. 도둑 클래스의 인스턴스 필드에는 역직렬화 도중의 싱글턴의 참조가 담겨있게 된다.
    6. 도둑 클래스의 readResolve 메서드는 인스턴스 필드가 참조한 값을 정적 필드로 복사한다.
    7. 싱글턴은 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다.
    8. 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 ClassCastException 이 발생한다.

### Enum

* 필드를 transient로 선언하여 문제를 해결할 수 도 있지만, 클래스를 원소 하나짜리 열거 타입으로 바꾸는 게 더 좋은 해결 방법이다.
* 열거 타입을 사용하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다. (AccessibleObject.setAccessible 같은 특권 메서드는 제외)

### readResolve 주의점

* 컴파일타임에 어떤 인스턴스가 있는지 알 수 없는 상황이라면, Enum을 사용할 수 없으므로 readResolve 방식을 사용해야 한다.
* final 클래스에선 readResolve 메서드가 private이어야 한다.
* readResolve 메서드가 protected/public 이면서 하위 클래스에서 재정의 하지 않았다면, 하위 클래스의 인스턴스를 역직렬화할 때 상위 클래스의 인스턴스를 생성하므로 ClassCastException이 발생할 수 있다.

## item 90) 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

### 직렬화 프록시 패턴

* Serializable의 버그와 보안 문제 위험을 줄여주는 기법

### 직렬화 프록시 구현 방법

* 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스(직렬화 프록시 클래스)를 설계해 private static으로 선언한다.
* 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받는다.
* 중첩 클래스의 생성자는 단순히 인수로 넘어온 인스턴스를 복사한다. (일관성 검사 및 방어적 복사 필요 X)
* 바깥 클래스와 직렬화 프록시 모두 Serializable을 선언한다.
*   다음은 직렬화 프록시 클래스의 예시이다.

    ```java
    class Period implements Serializable {
        private final Date start;
        private final Date end;

        public Period(Date start, Date end) {
            this.start = start;
            this.end = end;
        }

    		// 직렬화 프록시 클래스
    		private static class SerializationProxy implements Serializable {
    				private final Date start;
    				private final Date end;
    				
    				public SerializationProxy(Period p) {
    				this.start = p.start;
    				this.end = p.end;
    		    }
    				
    				private static final long serialVersionUID = 234098243823485285L;
    		}
    }
    ```

#### writeReplace

* 직렬화 프록시 사용하는 바깥 클래스에 **writeReplace** 메서드를 작성한다.
* 직렬화가 이뤄지기 전 바깥 클래스 인스턴스 대신 프록시 인스턴스를 반환하도록 한다.

```java
private Object writeReplace() {
  return new SerializationProxy(this);
}
```

#### readObject

* 직렬화 프록시 사용하는 바깥 클래스에 **readObject** 메서드를 작성한다.
* 불변식을 훼손하려 공격이 들어와도 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.

```java
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
}
```

#### readResolve

* 역직렬화 시 직렬화 시스템이 프록시 인스턴스를 바깥 클래스의 인스턴스로 변환하게 해줘야 한다.
* 일반 인스턴스 생성할 때와 똑같은 생성자, 정적 팩토리 등을 사용해 역직렬화된 인스턴스를 생성한다.

```java
private Object readResolve() {
    return new Period(start, end);
}
```

### 장점

* 가짜 바이트 스트림 공격이나 내부 필드 탈취 공격을 프록시 수준에서 차단할 수 있다.
* 필드를 final로 선언해도 되므로 직렬화 시스템에서 진정한 불변으로 만들 수 있다.
* 역직렬화 시 유효성 검사를 수행하지 않아도 된다.
* 역직렬화한 인스턴스와 원래의 직렬화된 클래스가 달라도 정상적으로 동작한다.
  * 원소 개수에 따라 다른 타입을 사용하는 EnumSet의 예시
    * 원소가 64개 이하면 RegularEnumSet을 그보다 크면 JumboEnumSet을 반환한다.
    * 만약 64개짜리 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개를 추가하고 역직렬화하면, 처음 직렬화된 것은 RegularEnumSet이지만 역직렬화될 때는 JumboEnumSet 인스턴스를 반환할 것이다.

### 한계점

* 클라이언트가 마음대로 확장할 수 있는 클래스에는 적용할 수 없다.
* 객체 그래프에 순환이 있는 클래스에 적용할 수 없다. 실제 객체는 아직 만들어지지 않아 readResolve 내에서 메서드 호출 시 ClassCastException이 발생한다.
* 방어적 복사보다는 속도가 느리다.
