# 자바 정의와 동작방식

## 자바의 정의

### 자바란?

#### WORA 사상 기반의 범용 프로그래밍 언어

* Write Once, Run Anywhere&#x20;
* 어느 하드웨어이든 상관 없이 자바로 작성된 프로그램을 실행시킬 수 있도록 한다.

#### 순수한 Object Oriented Language가 아니다

* 객체 지향의 특징은 추상화, 다형성, 캡슐화, 상속이 있다.
* 순수 객체 지향 언어가 되려면 모든 사전 정의 데이터타입과 사용자 정의 타입은 객체여야 하지만 **원시 타입, 정적 메서드, 래퍼 클래스는 객체가 아니다** !
* 일부 절차적인 요소가 존재한다.
* 객체에 대한 모든 작업은 객체 스스로 정해야 하지만 DTO 등은 외부로 입력받아 Getter/Setter를 사용하는 것 뿐인 수동적인 객체이다.

### Java 아키텍처

<figure><img src="../../.gitbook/assets/image (19).png" alt="" width="347"><figcaption></figcaption></figure>

* Java 플랫폼
  * JDK보다 넓은 의미로 **Java 개발 및 실행 환경**을 의미
  * SE, EE, ME 등 JDK를 구현한 제품
* JDK (Java Development Kit)
  * Java 프로그램 실행 및 개발 환경 제공
  * JRE + Development tools 로 구성된다
* JRE (Java Runtime Enviroment)
  * 자바 실행 환경으로, JVM + Library 로 구성된다.
* JVM
  * 프로그램을 실행하는 자바 가상 머신
  * Java 바이트코드를 기계어로 변환하고 실행
  * 자바 컴파일러와는 다른 모듈이다.
  * 다양한 플랫폼 위에서 자바를 동작시킬 수 있게 하는 중요한 역할을 한다.
  * 논리적인 개념, 여러 모듈의 결합체
  * 대표적인 역할
    * 클래스 로딩, GC 등 메모리 관리, 스레드 관리, 예외 처리

> 모듈: 코드,데이터를 그룹화하여 재사용이 가능한 정적인 단위
>
> 컴포넌트: 독립적으로 실행될 수 있는 소프트웨어 단위

## 자바 동작 방식

### 컴파일러와 인터프리터

* 컴파일러 방식
  * 프로그래밍 언어로 작성된 코드를 타겟 언어로 변환(번역)하는 프로그램
  * 주로 High-Level 언어를 Low-Level 언어(assembly, object code, machine code 등)로 변환
* 인터프리터 방식
  * 읽은 코드 및 해당 명령을 직접 분석/실행하는 프로그램
  * 세가지 방식을 사용할 수 있다.
    * 코드 구문을 분석하고 동작하는 것을 직접 모두 수행
    * 코드를 object code로 변환하고 즉시 실행
    * 컴파일러에 의해 생성된 바이트 코드를 명시적으로 실행
* 자바는 인터프리터의 두번째, 세번째 방식을 혼합해 사용하고 있어 컴파일  언어로 분류된다.

### Java 실행을 위한 준비

* 플랫폼에 의존적이지 않게, 그리고 효율적으로 컴파일 하기 위해 **자바 컴파일러(javac)**와 **JVM**으로 분리한다.
* 자바 컴파일러는 **소스 코드를 바이트코드로 변환**하는 역할을 담당한다.
* JVM에서는 클래스 로더가 로딩되어 **실행에 필요한 것을 JVM 내부에 저장**하고, 변환된 **바이트코드를 실행 엔진이 해석하고 실행**한다.
* 자바 컴파일러는 JVM 요소가 아닌 JDK의 일부이다.

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption><p>자바 컴파일러의 동작</p></figcaption></figure>

### JVM 동작 방식

* 컴파일된 바이트 코드 형태의 클래스들이 JVM에 전달되면, 아래 그림과 같은 흐름으로 코드가 실행되게 된다.
* 이제부터 Class Loader, Runtime Data Areas, Execution Engine 등에 대해 하나씩 살펴보도록 하자 🐾&#x20;

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

#### Class Loaders

* 한번에 모든 클래스를 로딩하는 것이 아닌, **필요할 때마다** 클래스나 리소스를 **로딩**한다.
* 바이트코드를 Loading → Linking → Initialization 의 순서로 처리한다.

#### [Runtime Data Areas](jvm-memory.md#jvm)

* 앱 실행을 위해 사용되는 JVM **메모리 영역**
* 구성
  * PC Registers
    * 스레드 별로 생성되며 실행 중인 명령(오프셋)을 저장하는 영역
  * Java Virtual Machine Stacks (Stack Area, Java Stack)
    * 스레드 별로 생성되며 메서드 실행 관련 정보를 저장하는 영역 (메서드가 호출되면 스택에 프레임 단위로 쌓이게 된다)
  * Heap
    * JVM 실행 시 생성되며 모든 객체 인스턴스/배열에 대한 메모리가 할당되는 영역
    * GC가 도는 영역은 보통 Heap영역뿐이다.
  * Method Area
    * JVM 실행 시 생성되며 클래스의 구조나 정보에 대한 정보가 메모리에 할당
  * Native Method Stacks
    * 스레드 별로 생성되며 네이티브 코드 실행에 관련 정보를 저장하는 영역

#### Execution Engine

* 메모리 영역에 있는 데이터를 가져와 해당하는 작업 수행
* JIT 컴파일러, GC, 인터프리터 존재
* 자주쓰는 메서드이면 미리 네이티브 코드(기계어)로 변환해두고 사용한다.

#### JNI (Java Native Interface)

* JVM과 네이티브 라이브러리 간 이진 호환성을 위한 인터페이스
* 네이티브 언어로 작성된 메서드 호출, 데이터 전달과 메모리 관리 등 수행

#### Native Libraries

* 네이티브 메서드의 구현체를 포함한 플랫폼별 라이브러리

## 클래스 로더와 클래스 로딩

### 클래스 로더

* 런타임에 바이트 코드를 JVM 메모리로 동적 로딩한다.
* Loading → Linking → Initialization 의 순서를 거친다.
* 계층 구조를 가진다.

#### Loading&#x20;

* JVM이 필요로 하는 클래스 파일 로드

#### Linking

* 로드된 클래스의 verify, prepare, resolve 작업 수행
* 해당 바이트코드의 결합해 실행할 수 있도록 하는 프로세스
* 로딩된 바이트코드의 유효성을 검증
* 선언된 스태틱 필드를 초기화하고 필요한 메모리를 할당
* 심볼릭 레퍼런스를 실제 참조, 프로세스 등으로 변환하는 작업
  *   심볼릭 레퍼런스의 검증은 링킹보다 나중에 처리될 수 있음

      > 심폴릭 레퍼런스: 자바 바이트코드에서 클래스, 인터페이스, 필드 등 참조하는 다른 요소를 표현한 방식
* 링킹은 새로운 자료구조의 할당을 포함하기 때문에 OutOfMemoryError가 발생할 수 있음

#### Initializing

* 초기화 메서드를 실행하며 클래스, 인터페이스, 정적 필드 등을 초기화한다.

#### 클래스 로더 종류

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

* Bootstrap ClassLoader
  * 최상의 클래스 로더
  * base 모듈을 로딩한다.
  * 네이티브 코드(c, c++)로 작성되어있어 ArrayList등의 class loader를 확인하려 하면 null로 보인다고 한다.
* Platform ClassLoader (Extension ClassLoader)
  * JAVA SE platform의 모든 API/클래스를 로딩한다.
* System ClassLoader (Application ClassLoader)
  * Java 어플리케이션 레벨 클래스를 로딩한다.
  * 클래스패스, 모듈패스에 있는 클래스들을 로딩한다.

### 클래스 로딩

* 특정 이름(FQCN)을 가진 클래스의 바이트코드를 찾아 클래스를 만드는 프로세스

> FQCN: Fully Qualified Class Name

* 최하위 클래스 로더부터 클래스를 찾고, loadClass메서드를 통해 클래스로딩을 수행한다.
* 로딩되지 않은 클래스는 바로 로딩하지 않고 상위 클래스 로더에 위임한 후 찾지 못하면 findClass 메서드를 호출해 클래스를 로딩한다.
* 최하위 클래스 로더까지 로딩을 실패하면 아래 두 예외가 발생하게 된다.
  * ClassNotFoundException: 런타임에 FQCN 에 해당하는 클래스가 존재하지 않을 때 발생, 일반적으로 클래스명을 사용해 리플렉션할 때 발생
  * NoClassDefFoundError: 클래스는 존재하지만 로드가 되지 않는 경우 발생, 일반적으로 static 블록 실행이나 static 변수 초기화 시 예외가 발생한 상황에서 에러 발생
* 모든 객체는 Monitor를 하나씩 가지고 있어 wait() 등을 사용 가능

## 바이트코드와 코드캐시

### 언어의 수준

* Machine code
  * CPU를 제어하는 기계어로 된 명령 코드
  * 각 명령 코드는 CPU 레지스터 또는 메모리에 있는 데이터에 대한 로딩, 저장, 연산 등 수행 일반 프로그래머가 액세스할 수 없는 특정 CPU 내부 코드
* Binary code
  * 두 개의 기호(일반적으로 0, 1)를 사용해 텍스트 또는 프로세스 명령을 나타내는 코드
  * 기계어는 binary code로 구성되어있다.
* Object code
  * 일반적으로 컴파일러에 의해 생성된 중간 언어로 작성된 명령 코드
  * object code는 Binary code이지만 기계어는 아니다.

### 코드 캐시

* JVM이 Java 바이트코드를 컴파일한 네이티브 코드를 저장하는 메모리 영역
* 실행 가능한 네이티브 코드의 블록을 \`nmethod\`이라고 함
* 미할당된 영역에 할당되며 힙(자료구조) 형태로 JIT 컴파일러가 코드 캐시 영역을 가장 많이 사용하게 됨\


### AOT 컴파일러

* Ahead of Time - **실행 전 바이트코드를 전부 컴파일**해두는 방식
* Java 바이트코드(.class 파일)를 AOT 컴파일러(jaotc) 통해 컴파일 하는 방식
* 이 기능을 수행하는 jaotc(Graal 컴파일러)가 JDK 17에서 제거되었지만 GraalVM을 별도로 추가하면 사용할 수 있다.
* AOT 컴파일러를 구현한 것이 Graal Compiler이고, 이를 사용한 JVM이 GraalVM이다.
* JVM 워밍업(부팅 시간)을 단축해 서버 실행 속도를 단축시키는 것이 가장 큰 목적
  * 서버 실행 속도가 빠르면 클라우드 환경에 컨테이너 이미지를 배포할 때 빠르게 떠서 요청량을 감당할 수 있으므로 적합하다
* 플랫폼에 의존적이고, 런타임에 알 수 있는 정보를 획득할 수 없으므로 이부분에서는 최적화가 불가능하다.

### JIT 컴파일러

*   Just In Time 컴파일러

    <figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
* hot code인지 확인한 후 JIT 컴파일러로 가거나 Interpreter로 간다.
* C1과 C2 컴파일러 모드로 구성되어 있다.
  * Java 7 버전 이후에는 두 모드를 혼합하여 C1 모드로 웜업 후 C2로 전환 (7 이전 버전에선 택일)&#x20;
* C1 (Client Compiler)
  * 런타임에 바이트코드를 기계어로 컴파일
  * 빠른 시작과 견고한 최적화가 필요한 앱에서 사용됨 (GUI 앱 등)&#x20;
* C2 (Server Compiler)
  * 변환된 기계어를 분석하여 C1보다 오래 걸리지만 더욱 최적화된 네이티브 코드로 컴파일
  * 오랫동안 실행을 하는 서버 앱 용도
* 플랫폼 확장성이 좋으며, 런타임 정보를 포함해 최적화 할 수 있다.
