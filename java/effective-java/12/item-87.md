# item 87) 커스텀 직렬화 형태를 고려해보라

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
