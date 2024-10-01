# Muitithreaded Programming

### Single and Multithreaded Processes

* 프로세스마다 코드, 데이터, 파일 영역에 대한 데이터를 갖고 있는데, 각각 고유한 메모리 영역을 가지므로 다른 프로세스의 영역에 접근하기 어렵다.
* 멀티스레드로 한 프로세스를 여러 개의 thread로 구성하면 **코드, 데이터, 파일 영역을 공유**하여 자유롭게 사용할 수 있다
* 스레드가 여러 개일 경우 스레드 간의 context switching을 할 수 있어 용이하다 (메모리 영역 참조와 캐시를 그대로 유지 할 수 있다)
  * 프로세스 간의 context switching을 하려면 MMU가 바라보는 주소값도 변경해야 하고 TLB도 비워줘야 하므로 성능이 느리다

<figure><img src="../../.gitbook/assets/Untitled 7.png" alt=""><figcaption></figcaption></figure>

### Thread 종류

#### Hardware Thread

* OS 관점에서는 가상의 코어
* 하나의 코어에 여러 하드웨어 스레드가 존재할 수 있다.
* OS는 하드웨어 스레드의 개수 만큼 코어가 있다고 간주하여 OS 레벨의 스레드들을 스케줄링한다.

#### OS Thread

* CPU에서 실제로 실행되는 스레드 단위
* OS 스레드의 컨텍스트 스위칭은 커널이 개입하여 오버헤드가 발생한다.
* 사용자 코드와 커널 코드 모두 OS 스레드에서 실행된다.
* 커널 스레드, 네이티브 스레드, OS Level 스레드 등으로도 불린다.

#### User Thread

* 스레드 개념을 프로그래밍 레벨에서 추상화한 것
* 자바에서 Thread.start()로 스레드를 실행시킬 때, 이 유저 스레드가 생성되어 실행된다.
* 유저 스레드가 CPU상에서 실행되려면 OS 스레드와 반드시 연결되어야 한다.

### Thread Mapping

* 유저 스레드와 OS 스레드가 매핑되는 경우는 1:1, 1:N, N:M 세가지가 있다.
* One to One
  * 하나의 유저 스레드가 하나의 OS 스레드와 매핑된다.
  * 스레드 관리를 OS에 위임한다.
  * 스케줄링을 커널이 수행한다.
  * 멀티 코어를 활용할 수 있다.
  * 한 스레드가 블락되어도 다른 스레드는 잘 동작한다.
  * race condition 발생 가능
  * 한정된 자원이므로 무한 개로 생성 불가능
  * 현재 Java는 이 모델에 속한다.
* Many to One
  * 여러개의 유저 스레드가 하나의 OS 스레드와 매핑된다.
  * 유저 스레드 간의 스위칭이 빠르다.
  * race condition 발생할 가능성이 적으며, 그마저도 유저 스레드 레벨에서 발생한다.
  * 멀티코어를 활용할 수 없다.
  * 한 스레드가 블락되면 모든 스레드가 블락된다.
  * 커널 스레드를 지원하지 않는 시스템에서 사용된다.
* Many to Many
  * 여러개의 유저 스레드가 여러개의 OS 스레드와 매핑된다.
  * 운영 체제가 충분한 개수의 커널 스레드를 생성하도록 허용한다.
* 아래 그림은 간단한 예시이며, 옛날 자료라 Java가 Many to One으로 구성되어있지만, 현재 기준으로 One to One이 맞다.

<figure><img src="../../.gitbook/assets/Untitled 8.png" alt=""><figcaption></figcaption></figure>

#### Java VS Go

[https://medium.com/@genchilu/javas-thread-model-and-golang-goroutine-f1325ca2df0c](https://medium.com/@genchilu/javas-thread-model-and-golang-goroutine-f1325ca2df0c)



