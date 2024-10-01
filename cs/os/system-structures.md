# System Structures

<figure><img src="../../.gitbook/assets/Untitled (6).png" alt="" width="375"><figcaption></figcaption></figure>

### 유저 모드

* 일반적으로 프로그램을 실행 시킬 때 이 모드에서 실행된다.

#### 인터럽트

* 전원에 문제가 생겼을 때, I/O 작업이 완료됐을 때, CPU 할당 시간이 다 됐을 때, 0으로 나눴을 때, 잘못된 메모리 공간에 접근할 때 발생한다.
* CPU는 인터럽트 발생 시 즉시 시스템 콜을 호출해 커널 코드를 커널 모드에서 실행한다.

#### 시스템 콜

* 프로그램이 OS 커널이 제공하는 서비스를 이용하고 싶을 때 시스템 콜을 호출해 실행
* 프로세스/스레드, 파일 I/O, 소켓, 장치, 프로세스 통신 등에 관련된 작업을 처리한다.
* 운영체제 단에서 제공해주는 시스템 콜의 종류가 있다.
* 하드웨어나 시스템 관련 기능은 반드시 시스템 콜을 통해 사용 가능하다.
* 예를 들어 Java에서 Thread를 시작하는 start() 메서드는 내부적으로 JNI 코드를 호출하는데, 이 JNI 코드가 OS 시스템 콜 중 하나인 clone()을 호출한다.



### 커널 모드

#### 커널

* 시스템의 전반적인 것들을 관리하고 감독한다.
* 하드웨어와 관련된 작업을 직접 수행한다.

#### 커널 모드의 필요성

* 프로그램의 현재 CPU 상태를 저장하고 인터럽트나 시스템 콜을 처리한다.
* 모든 처리가 완료되면 중단되었던 프로그램의 CPU 상태를 복원하거나, 다른 프로그램의 상태를 복원한다.
* 각 프로세스가 시스템 자원들을 직접 접근할 수 있다면 복잡하게 점유하다가 전체 컴퓨터 시스템이 붕괴될 수 있다.

#### 커널 모드가 하는 일

* Process Management
  * 프로세스를 생성, 삭제, 중단, 재개
  * 프로세스 간 synchronization(동기화), IPC(Inter Process Communication) 관리
* Memory Management
  * 어떤 부분의 메모리가 누구에게 사용되는지 추적
  * 메모리 공간 사용 가능할 때 어떤 프로세스 로드할 지 결정
  * 필요에 따라 메모리 공간 할당, 해제
* I/O System Management
* Secondary Storage Management
* File System Management
  * 파일/디렉토리 생성 및 삭제
  * 파일을 secondary storage에 매핑
  * 파일을 비휘발성 저장소에 백업
* Networking
* Protection System
* Command-Interpreter System

#### Inter Process Communication

* 커널에서 제공되는 프로세스 간 통신을 돕는 방법
* 종류: 익명 PIPE, Named PIPE, Message Queue, 공유 메모리, 메모리 맵

