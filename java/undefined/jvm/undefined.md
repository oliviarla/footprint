# 클래스 로더

## 클래스 로더 동작 과정

* 런타임에 바이트 코드를 JVM 메모리로 동적 로딩한다.
* Loading → Linking → Initialization 의 순서를 거친다.
* 계층 구조를 가진다.

### Loading&#x20;

* JVM이 필요로 하는 클래스 파일을 로드하는 과정이다.
* 메서드 영역에 Fully Qualified Class Name와 클래스/인터페이스/Enum 정보와 메서드, 변수를 저장한다.
* 로딩이 끝나면 해당 클래스 타입의 **Class 객체를 생성해** **힙 영역에 저장**한다.
* bootstrap -> platform -> system 순으로 클래스 로더에서 클래스를 찾고, 모든 클래스 로더로부터 클래스를 찾을 수 없으면 ClassNotFoundException이 발생한다.

### Linking

* 로드된 클래스의 **verify, prepare, resolve** 작업을 수행한다.
  * **verify**: .class 파일 형식이 유효한지 체크
  * **prepare**: 클래스 변수와 기본값에 필요한 메모리를 준비
  * **resolve**: 심볼릭 메모리 레퍼런스를 메서드 영역에 있는 실제 레퍼런스로 교체 (optional)
* 해당 바이트코드를 결합해 실행할 수 있도록 하는 프로세스
* 로딩된 바이트코드의 유효성을 검증
* 선언된 스태틱 필드를 초기화하고 필요한 메모리를 할당
* 심볼릭 레퍼런스를 실제 참조, 프로세스 등으로 변환하는 작업 수행
  *   심볼릭 레퍼런스의 검증은 링킹보다 나중에 처리될 수 있음

      > 심폴릭 레퍼런스: 자바 바이트코드에서 클래스, 인터페이스, 필드 등 참조하는 다른 요소를 표현한 방식
* 링킹은 새로운 자료구조의 할당을 포함하기 때문에 OutOfMemoryError가 발생할 수 있음

### Initializing

* 초기화 메서드를 실행하며 **클래스, 인터페이스, 정적(static) 변수 등을 초기화**한다.
* **static 블럭**이 있다면 여기에서 **실행**된다.

## 클래스 로더 종류

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

### Bootstrap ClassLoader

* 최상위 우선순위를 가진 클래스 로더
* java.base 모듈의 클래스들을 로딩한다.

### Platform ClassLoader (Extension ClassLoader)

* JAVA SE Platform의 모든 API/클래스를 로딩한다.

### System ClassLoader (Application ClassLoader)

* Java 어플리케이션 레벨 클래스를 로딩한다.
* 애플리케이션 실행 시 환경 변수로 주는 classpath, module-path 경로에서 클래스들을 로딩한다.

### 클래스 로더 접근

* 아래와 같이 클래스로더에 접근할 수 있다.
* bootstrap 클래스 로더는 네이티브 코드(c, c++)로 작성되어있어 자바에서 참조 불가능하다.

```java
ClassLoader appClassLoader = App.Class.getClassLoader();
ClassLoader platformClassLoader = classLoader.getParent();
ClassLoader bootstrapClassLoader = classLoader.getParent().getParent(); //null
```

## 클래스 로딩

* 특정 이름(FQCN)을 가진 클래스의 바이트코드를 찾아 클래스를 만드는 프로세스

> FQCN: Fully Qualified Class Name

* 최하위 클래스 로더부터 클래스를 찾고, loadClass메서드를 통해 클래스로딩을 수행한다.
* 로딩되지 않은 클래스는 바로 로딩하지 않고 상위 클래스 로더에 위임한 후 찾지 못하면 findClass 메서드를 호출해 클래스를 로딩한다.
* 최하위 클래스 로더까지 로딩을 실패하면 아래 두 예외가 발생하게 된다.
  * ClassNotFoundException: 런타임에 FQCN 에 해당하는 클래스가 존재하지 않을 때 발생, 일반적으로 클래스명을 사용해 리플렉션할 때 발생
  * NoClassDefFoundError: 클래스는 존재하지만 로드가 되지 않는 경우 발생, 일반적으로 static 블록 실행이나 static 변수 초기화 시 예외가 발생한 상황에서 에러 발생
* 모든 객체는 Monitor를 하나씩 가지고 있어 wait() 등의 모니터 메서드를 사용할 수 있다.
