# Java 언어의 특징

#### WORA 사상 기반의 범용 프로그래밍 언어

* Write Once, Run Anywhere&#x20;
* 자바 응용프로그램은 운영체제나 하드웨어가 아닌 JVM하고만 통신하므로 어느 하드웨어이든 상관 없이 자바로 작성된 프로그램을 실행시킬 수 있도록 한다.

#### 순수한 Object Oriented Language인가?

* 객체 지향의 특징은 추상화, 다형성, 캡슐화, 상속이 있다. 자바는 상속, 캡슐화, 다형성이 잘 적용된 언어이다.
* 다만, 순수 객체 지향 언어가 되려면 모든 사전 정의 데이터타입과 사용자 정의 타입은 객체여야 하는데 자바의 **원시 타입, 정적 메서드, 래퍼 클래스는 객체가 아니다.** 이외에도 일부 절차적인 요소가 존재한다.
* 객체에 대한 모든 작업은 객체 스스로 정해야 하지만 DTO를 사용하는 방식은 외부로 입력받아 Getter/Setter를 사용하는 것 뿐인 수동적인 객체이다.

#### 자동 메모리 관리

GC(Garbage Collection)가 존재하여 자동으로 메모리를 관리해준다. 이로 인해 직접 메모리를 할당하고 해제하는 C/C++에 비해 편리하다.

#### 네트워크와 분산 처리 지원

다양한 네트워크 프로그래밍 라이브러리를 제공한다.

#### 멀티스레드 지원

시스템과 관계없이 멀티스레드 프로그램을 구현 할 수 있으며, 여러 스레드에 대한 스케줄링을 자바 인터프리터가 담당한다.

#### 동적 로딩 지원

실행 시 모든 클래스가 로딩되는 대신에, 필요한 시점에 클래스를 로딩해 사용하므로 일부 클래스가 변경되어도 컴파일을 다시 하지 않아도 된다.&#x20;
