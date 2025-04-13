# 싱글톤 패턴

## 접근

* 객체를 만들어 사용할 때 인스턴스가 2개 이상이면 프로그램에 이상이 생기는 경우가 발생할 수 있다.
* 이러한 경우 특정 클래스에 객체 인스턴스를 하나만 만들 수 있도록 하는 패턴을 사용해야 한다.
* 전역 변수를 사용하는 것과 같이 객체 인스턴스를 어디서든 접근할 수 있도록 하고, 해당 객체가 필요한 상황에 객체 인스턴스를 생성하면 된다.
  * 전역변수로 싱글톤 패턴을 구현할 경우, 언어마다 특징이 달라 외부 코드에서 덮어쓰여질 가능성이 있으므로 조심해서 사용해야 한다.

## 개념

* 클래스 인스턴스를 하나만 만들고, 그 인스턴스로의 전역 접근을 제공하는 디자인 패턴이다.
* 클래스 내부에서 하나뿐인 인스턴스를 관리한다. 다른 클래스에서 인스턴스 생성이 불가능하도록 막고, 클래스에 요청해야만 인스턴스를 얻을 수 있다.

## 장단점

* 장점
  * 하나의 객체를 전역적으로 접근하는 것을 보장한다.
  * 초기화가 한 번만 이루어진다.
* 단점
  * 싱글톤 객체는 사용하는 객체와 강하게 결합되어, 싱글톤 클래스에 변경이 발생했을 때 사용하는 쪽 클래스도 변경되어야 한다.
  * 객체 고유의 행동에 대한 책임, 객체를 싱글톤으로 유지하는 책임을 동시에 가지므로 단일 책임 원칙을 위반한다.
  * 테스트 코드 작성 시 Mock 객체를 사용하기 어렵다.

## 주의점

### 클래스로더

* 클래스 로더가 여러 개라면 클래스 로더마다 서로 다른 네임스페이스를 정의하므로 같은 클래스를 여러 번 로딩할 수 있어 싱글톤 인스턴스가 여러 개 만들어질 수 있다. 따라서 클래스 로더가 여러 개인 상황이라면 주의해야 한다.

### 리플렉션, 직렬화/역직렬화

* 리플렉션을 통해 클래스 생성자에 접근하면 객체가 하나라는 원칙이 깨지게 된다. 이 때에는 생성자에서 예외를 발생하면 된다.
* 싱글톤 객체를 직렬화한 후 역직렬화하면 새로운 객체가 생성된다.
* 자세한 내용은 여기를 참고한다.[item-89-readresolve.md](../../java/effective-java/12/item-89-readresolve.md "mention")

## 사용 방법

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="248"><figcaption></figcaption></figure>

* 외부에 노출되는 생성자가 없도록 모두 private 접근자로 둔다.
* getInstance 정적 메서드에서 객체를 한 번만 생성하고, 생성된 객체를 필드에 저장해두어 재활용할 수 있도록 한다.
* 멀티 스레드 환경에서 동시에 uniqueInstance가 존재하지 않을 때 getInstance 메서드에 접근하면 각기 다른 객체를 생성해 사용하게 되므로 문제가 발생한다.
*   이를 해결하려면 아래 세 가지 방법 중 하나를 사용하면 된다.

    * getInstance 메서드에 `synchrinozed` 키워드를 붙여 동시에 여러 스레드에서 호출하지 못하게 한다.
    * 인스턴스를 클래스 초기화 시점에 생성하도록 한다.
    * DCL(Double-Checked Locking) 방식을 통해 객체 생성 전 인스턴스가 있는지 확인하고, `synchronized` 블록을 통해 싱글톤 클래스 자체에 락을 걸어 객체 생성을 동시에 수행되지 않도록 한다. 이를 통해 처음 객체를 생성할 때에만 동기화가 이뤄지도록 한다.

    ```java
    public class Singleton {
        private volatile static Singleton uniqueInstance;

        public static Singleton getInstance() {
            if (uniqueInstance == null) {
                synchronized (Singleton.class) {
                    if (uniqueInstance == null) {
                        uniqueInstance = new Singleton();
                    }
                }
            }
            return uniqueInstance;
        }
    }
    ```

## 예시

* Spring Session에서는 UUID를 비롯한 다양한 방식을 통해 세션 아이디를 생성할 수 있다.
* 이 중 가장 기본적이고 널리 사용되는 UUID 방식으로 세션 아이디를 생성하도록 한 UuidSessionIdGenerator는 싱글톤으로 동작하도록 구현되었다.

```java
public final class UuidSessionIdGenerator implements SessionIdGenerator {

	private static final UuidSessionIdGenerator INSTANCE = new UuidSessionIdGenerator();

	private UuidSessionIdGenerator() {
	}

	@Override
	@NonNull
	public String generate() {
		return UUID.randomUUID().toString();
	}

	/**
	 * Returns the singleton instance of {@link UuidSessionIdGenerator}.
	 * @return the singleton instance of {@link UuidSessionIdGenerator}
	 */
	public static UuidSessionIdGenerator getInstance() {
		return INSTANCE;
	}

}
```

* Spring Cache에서는 Null 저장이 불가능한 캐시 솔루션을 다루기 위해 NullValue라는 클래스를 Null 대신 저장하도록 유도하고 있다. 여기서는 싱글톤 객체 필드에 직접 접근하도록 하고 있어 전역 변수처럼 사용하고 있다.

```java
public final class NullValue implements Serializable {

    public static final Object INSTANCE = new NullValue();
    // ...
}
```
