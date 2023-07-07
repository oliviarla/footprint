# Process

### Process State

<figure><img src="../../.gitbook/assets/Untitled 4 (1).png" alt=""><figcaption></figcaption></figure>

* **new**
  * 프로세스가 처음 생성된 상태
* **ready**
  * 프로세스가 실행할 준비가 되었고 CPU의 자원을 받기 위해 기다리는 상태
  * 이 ready 상태의 프로세스들을 [큐](process.md#process-scheduling-queues)로 관리한다.
* **running**
  * CPU로부터 자원을 할당받아 프로세스가 실행되는 상태
  * 할당된 time quantum을 다 사용했는데 프로세스 실행이 종료되지 않았을 때에는 다시 ready 상태로 돌아가 자원 할당을 기다려야 한다.
* **waiting**
  * 프로세스에서 I/O 작업이 필요하거나 critical section에 접근해야 할 경우 waiting 상태로 변경된다.
  * CPU 낭비를 방지하기 위해 waiting 중에는 다른 프로세스가 실행된다.
  * I/O 작업이 완료되거나 lock을 획득할 수 있으면 ready 상태로 변경되어 기다리다가 CPU에 할당되면 running 상태가 된다.
* **terminated**
  * 프로세스가 종료된 상태

#### scheduler

* CPU가 항상 바쁜 상태로 유지되도록 프로세스를 선택한다
* 실행하던 프로세스가 waiting / ready / terminated 등으로 변할 경우 다른 프로세스를 CPU에 할당해야 한다.
* 이 때 어떤 프로세스를 CPU에 할당할 지 스케쥴러가 확인하게 된다.

#### dispatcher

* 스케줄러에 의해 선택된 프로세스가 CPU에서 실행될 수 있도록 한다.
* 커널 모드에서 context switching를 담당한다.
* 새로운 프로세스가 실행될 수 있도록 유저 모드로 전환해준다.
* 실행되어야 할 시작 위치로 프로세스를 이동시킨다.

### **Process Scheduling Queues**

* Job queue
  * 시스템 내의 모든 프로세스를 저장
  * 현재 쓰이지는 않음
* Ready queue
  * 실행 가능한 프로세스들이 자원 할당을 위해 메인 메모리에서 대기하는 큐
* Device queue
  * I/O device에서 작업을 처리할 때 대기하는 프로세스 큐

<figure><img src="../../.gitbook/assets/Untitled 5.png" alt=""><figcaption></figcaption></figure>

### Communication Models

* memory protection 때문에 프로세스 A에서 프로세스 B로 데이터를 직접 보낼 수 없다.
* **Message Passing**
  * 특정 프로세스에서 커널을 통해 다른 프로세스로 메시지 전송
  * 장점: 커널이 데이터를 옮기고 프로세스 간 동기화를 함
  * 단점: 데이터를 옮길 때 운영체제를 거치므로 데이터 양이 크면 성능 상 오버헤드가 커짐, 속도 느림
* **Shared Memory**
  * 프로세스 A와 프로세스 B가 공유하는 메모리 공간을 따로 할당
  * 장점: 공유 메모리를 만드는 과정만 커널이 하고, 나머지는 프로세스가 직접 메모리에 관여하므로 오버헤드가 줄어듦
  * 단점: 프로세스 안에서 데이터 동기화를 직접 조절해야 함, 데이터가 메일 박스에 있는지 계속 확인해야 함

### Producer-Consumer Problem

* 두 프로세스에서 데이터를 주고 받을 때 발생하는 문제
* 데이터를 옮길 때 bounded buffer(유한 버퍼)로 shared memory를 생성하여 옮기므로, bounded-buffer problem 이라고도 부른다.
* Producer: 데이터 생산 담당 프로세스, 큐에 데이터를 저장
* Consumer: 데이터 사용하는 프로세스, 큐에 저장된 데이터를 꺼내 사용
* 문제점
  * 제한된 용량을 가진 shared memory를 사용하게 되면 **큐가 꽉 차면 Producer가 대기해야 하고, 큐가 비어 있을 경우 Consumer가 대기해야 한다.**
  * 다수의 Producer, Consumer가 있을 경우 같은 파일을 덮어쓰거나, 여러번 읽는 과정이 발생할 수 있다.
  * 프로세스마다 큐에 데이터가 있는지 확인하는 코드를 개발자가 직접 넣어줘야 한다. (운영체제 커널의 큐를 사용하면 운영체제가 알아서 관리)
* 해결법
  * Semaphore
  * Monitor
  * circular queue 구현 방식을 사용해, 데이터 입출력을 in/out 변수(포인터)로 관리한다.



참고 자료

* [https://www.baeldung.com/cs/bounded-buffer-problem](https://www.baeldung.com/cs/bounded-buffer-problem)
