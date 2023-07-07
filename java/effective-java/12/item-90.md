# item 90) 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

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
