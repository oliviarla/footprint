# Atomic

자바 프로그램은 다음과 같이 동작한다.

1. CPU에서 작업을 처리하기 위해 데이터를 RAM으로부터 읽어 CPU Cache Memory에 복제해둔다.
2. CPU에서 작업을 처리하여 새로 업데이트된 CPU Cache Memory 데이터를 메인 메모리(RAM)에 덮어씌운다. (곧바로 메인 메모리에 업데이트 하지는 않는다.)

<figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

이 때 멀티 스레드 방식을 위해 여러 CPU를 사용할 경우 RAM과 각각의 CPU Cache Memory 데이터 정합성이 맞지 않을 수 있다.

Atomic 패키지 내에서 제공하는 클래스들은 자바 프로그램에서 발생할 수 있는 가시성 문제와 동시 접근 문제를 방지할 수 있다.

* 가시성 문제
  * 하나의 스레드에서 공유 자원을 수정했지만 다른 스레드에서 보이지 않을 때 발생하는 문제
* 동시 접근 문제
  * 여러 스레드에서 공유 자원에 동시에 접근할 때 가장 늦게 끝난 연산의 결과로 덮어씌워지는 문제

### AtomicLong

* long 자료형 데이터의 Atomicity을 보장하는 클래스이다.
* 내부 메서드는 AtomicInteger와 유사하다.

```java
// 생성자
AtomicLong al = new AtomicLong(); // 인자에 아무것도 넣지 않으면 초기값은 0이다.
AtomicLong al2 = new AtomicLong(3); // 초기값을 3으로 주어 생성한다.

// 조회
long result = al.get();

// 값 설정
al.set(10);

// 기존 값 조회 후 새로운 값으로 설정
long result = al.getAndSet(100);

// 기존 값이 예상하는 값과 동일하면 새로운 값으로 설정
boolean updated = al.compareAndSet(100, 101);
```



### AtomicIntegerArray

* int 배열 데이터의 Atomicity를 보장하는 클래스이다.

```java
// 생성자
AtomicIntegerArray aa = new AtomicIntegerArray(new int[]{1, 2, 3}); // 인자에 배열을 넣어 생성한다.
AtomicIntegerArray aa2 = new AtomicIntegerArray(3); // 배열의 길이를 입력받아 0으로 채워진 배열을 생성한다.

// 인덱스에 해당하는 값 조회
int result = aa.get(1);

// 인덱스 위치의 값 설정
aa.set(0, 10);

// 기존 값 조회 후 새로운 값으로 설정
int result = aa.getAndSet(0, 100);

// 기존 값 조회 후 기존 값에 람다식을 적용해 업데이트
int result = aa.getAndUpdate(0, i -> i * i);

// 기존 값이 예상하는 값과 동일하면 새로운 값으로 설정
boolean updated = aa.compareAndSet(0, 100, 101);
```

### AtomicReference

* 제네릭 데이터의 Atomicity를 보장하는 클래스이다.

```java
// 생성자
AtomicReference<String> ar = new AtomicReference<>(); // 객체를 null로 초기화한다.
AtomicReference<String> ar2 = new AtomicReference<>("hello"); // 초기값 객체를 주어 생성한다.

// 인덱스에 해당하는 값 조회
String result = ar.get();

// 인덱스 위치의 값 설정
ar.set("hi");

// 기존 값 조회 후 새로운 값으로 설정
String result = ar.getAndSet("hi hello");

// 기존 값이 예상하는 값과 동일하면 새로운 값으로 설정
boolean updated = ar.compareAndSet("hi hello", "succeed");
```
