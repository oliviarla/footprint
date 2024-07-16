# JVM

### JVM이란 <a href="#jvm" id="jvm"></a>

* Java Virtual Machine
* JVM이 OS에서 실행되면, Java 애플리케이션은 JVM하고만 상호작용하므로 OS에 종속되지 않는다. 대신, JVM은 OS에 종속적이다.
* 자바로 작성한 애플리케이션을 해당 운영체제가 이해할 수 있도록 변환하여 전달한다.
* 바이트 코드를 기계어로 변환하고 실행한다.
* 아래 그림과 같이 자바 컴파일러(javac)와는 다른 모듈이다.

<figure><img src="https://blog.kakaocdn.net/dn/beHaHI/btrKrqMW4rU/oU3cGpEkHTjJYqvmbnewM0/img.png" alt="" height="267" width="369"><figcaption><p>java code to machine language</p></figcaption></figure>

{% hint style="info" %}
바이트 코드

* JVM이 이해할 수 있는 기계어
* 바이트 코드의 각 명령어는 1바이트 크기의 Opcode와 추가 피연산자로 이루어져 있다.
{% endhint %}

### JVM 동작 방식

* 컴파일된 바이트 코드 형태의 클래스들이 JVM에 전달되면, 아래 그림과 같은 흐름으로 코드가 실행되게 된다.
* 내부 구성 요소인 Class Loader, Runtime Data Areas, Execution Engine에 대해서는 아래에서 간략히 설명한다.

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

#### Class Loaders

* 컴파일된 .class 파일에서 바이트 코드를 읽고 메모리에 저장한다.
* 한번에 모든 클래스를 로딩하는 것이 아닌, **필요할 때마다** 클래스나 리소스를 **로딩**한다.
* 바이트코드를 Loading → Linking → Initialization 의 순서로 처리한다.
  * Loading: 클래스를 읽어오는 과정
  * Linking: 레퍼런스를 연결하는 과정
  * Initialization: static 값들을 초기화하고 변수에 할당하는 과정

#### Runtime Data Areas (메모리)

* 앱 실행을 위해 사용되는 JVM **메모리 영역**
* 구성
  * PC(Program Counter) Registers
    * 스레드 별로 생성
    * **현재 실행 중인 명령(오프셋 / 스택 프레임)을 가리키는 포인터**를 저장하는 영역
  * Java Virtual Machine Stacks (Stack Area, Java Stack)
    * 스레드 별로 생성
    * **메서드 실행 관련 정보를 저장**하는 영역
    * 메서드가 호출되면 스택에 프레임 단위로 메서드 정보가 쌓이게 된다.
  * Heap
    * JVM 실행 시 생성
    * **모든 객체 인스턴스/배열에 대한 메모리**가 할당되는 영역
    * GC가 도는 영역은 보통 Heap 영역뿐이다.
  * Method
    * JVM 실행 시 생성
    * **클래스의 구조 등에 대한 정보**가 메모리에 할당
  * Native Method Stacks
    * 스레드 별로 생성
    * **네이티브 코드 실행에 관련된 정보**를 저장하는 영역

#### Execution Engine

* 메모리 영역에 있는 데이터를 가져와 해당하는 작업 수행
* JIT 컴파일러, 인터프리터, GC가 존재한다.
* 자주쓰이는 코드는 JIT 컴파일러를 통해 미리 네이티브 코드(기계어)로 변환해두고, 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용한다.
* GC는 더이상 참조되지 않는 객체를 모아 메모리를 해제한다.

#### JNI (Java Native Interface)

* JVM과 네이티브 라이브러리 간 이진 호환성을 위한 인터페이스
* 네이티브 언어로 작성된 메서드 호출, 데이터 전달과 메모리 관리 등 수행
* 실행 시 메모리의 Native Method Stacks에 정보가 저장된다.

#### Native Libraries

* 네이티브 메서드의 구현체를 포함한 플랫폼 별 라이브러리
