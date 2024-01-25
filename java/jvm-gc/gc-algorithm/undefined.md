# 가비지 컬렉터 이해하기

## Garbage Collection

### 개념

* Garbage란 더이상 참조되지 않는 메모리를 의미한다.
* Garbage Collection은 말그대로 더이상 참조되지 않는 메모리를 제거해주는 자동 메모리 관리의 한 형태이며, 프로그램의 성능에 영향을 미치게 된다.
* Garbage Collection은 Heap 영역을 관리하게 된다.
* 만약 GC가 제대로 동작하지 못한다면, 더이상 사용되지 않는 객체가 메모리를 계속해서 점유하게 되고, 점점 메모리 공간이 부족해지면 OutOfMemoryError로 프로그램이 종료될 수 있을 것이다.!
* 자바는 언어 자체에서 가비지 컬렉션을 제공하여 C 언어와 다르게 프로그래머가 메모리를 직접 할당하고 해제하는 수동 관리의 부담을 덜어준다.
* 일반적으로 네트워크 소켓, DB 핸들, 윈도우, 파일 디스크립터 등과 같은 리소스는 GC 처리되지 않는다.

### 장점

* 메모리 직접 할당/해제할 필요 없다.
* 메모리 누수를 자동으로 관리해주어 완벽하진 않지만 상당 부분 처리가 가능하다.
* 댕글링 포인터 핸들링으로 인한 오버헤드가 발생하지 않는다.
* 비즈니스 로직에 좀 더 초점을 맞출 수 있다.

### 단점

* 성능을 직접 handling 하고 싶어도 제어할 수 없다.
* 모든 객체에 대한 생성/삭제를 추적하므로 원래의 앱 사용 리소스(CPU)보다 더 많은 성능이 필요하다.
* 프로그래머가 사용하지 않는 객체를 해제하는 전용 CPU 스케줄링을 직접 제어할 수 없다.
* GC를 위한 메모리나 CPU가 더 사용될 수 있다.
* 일부 GC는 기능적으로 완벽하지 않아 런타임에 중지될 수도 있다.

### Garbage Collector

* 자동 메모리 관리, 즉 Garbage Collection을 수행한다.
* 힙 메모리를 분석해 사용하지 않는 객체를 식별해 삭제하는 프로세스이다.
* 어디선가 참조중인(포인터 유지) 객체는 사용되고 있는 객체이고, 아무곳에서도 참조되지 않는 객체는 사용되지 않는 객체이기 때문에 이를 확인해 데이터를 제거하고 메모리를 회수해 공간을 확보해준다.

## GC Process 종류

* 가장 기본적인 GC의 동작으로는 Mark, Sweep, Compact가 있다. 어렵지 않은 내용이니 차근차근 살펴보자.

#### **Mark**

* 참조 객체와 참조되지 않는 객체를 식별하는 프로세스이다.
* 모든 객체를 스캔하므로 시간이 많이 걸리는 프로세스가 될 수 있다.
* 참조되지 않는 객체를 제거하기 위해 필요한 선수 작업이다.

<figure><img src="../../../.gitbook/assets/image (18).png" alt="" width="563"><figcaption></figcaption></figure>

#### **Sweep**

* 참조되지 않아 마킹된(미사용) 객체를 제거하는 프로세스
* 새 객체를 할당할 수 있는 여유 공간 블록에 대한 참조를 메모리 할당자가 들고 있도록 한다.

<figure><img src="../../../.gitbook/assets/image (21).png" alt="" width="563"><figcaption></figcaption></figure>

#### **Compact**

* Mark, Sweep 작업으로 인해 생긴 메모리 단편화를 없애는 프로세스
* 남아있는 참조 객체를 이동해 메모리 압축 작업을 수행하면 새 메모리 할당이 훨씬 빨라진다.
* 이 때에도 새 객체를 할당할 수 있는 여유 공간 블록에 대한 참조를 메모리 할당자가 들고 있도록 한다.

<figure><img src="../../../.gitbook/assets/image (12).png" alt="" width="563"><figcaption></figcaption></figure>

## JVM Generations

* 많은 객체가 할당되면 compact, marking이 오래 걸려 GC 작업 시간이 증가하는 현상을 보완하기 위한 방법이다.
* 아래 그림과 같이 힙 영역을 **객체의 오래된 정도를 기준으로** 영역을 작게 나누어, 가장 최신의 객체는 `Young Generation`, 오래된 객체는 `Old Generation`, 프로그램이 죽을 때까지 사용되는 객체는 `Permenent Generation`에 담을 수 있다.
* 오래 쓰인 object를 담는 영역은 쭉 참조되는 경우가 많기 때문에 가끔 확인해서 비우고, 새로생긴 object를 담는 영역은 자주 비우도록 처리하면 GC가 도는 영역을 대폭 줄일 수 있다.

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **young generation**

* 새로운 객체가 할당되는 영역이며, 1개의 Eden 영역과 2개의 Survivors로 나뉜다.
* 오래 남은 객체는 `old generation` 영역으로 이동한다.
* `Minor GC`가 발생한다.

> `Minor GC`: eden 영역과 객체들이 있는 survival 영역을 돌며 참조되지 않는 객체를 제거하고, 비어있는 survival 영역으로 참조중인 객체를 모두 이동시키는 프로세스이다. STW가 발생한다.

> STW(Stop The World) 이벤트: 작업이 완료될 때 까지 모든 앱 스레드가 중지되는 이벤트이다. 동시 처리가 안될 때 어쩔 수 없이 모든 작업을 중단해야 하기 때문이다.

#### **eden**

* 인스턴스 생성 시 eden 영역에 할당 된다.
* 살아남을 객체만 마킹하는 작업을 수행한 후, survival 영역으로 이동한다.

#### **survival**

* from과 to 의 두 영역으로 나뉘며, 객체들을 from에서 to로 이동시키며 GC를 실행한다.
* survival 영역에서 왔다갔다 이동하며 설정된 임계값(threshold)을 넘긴 객체들은 Old 영역으로 이동한다.

### old generation

* 오랫동안 참조(사용)된 객체가 저장되는 영역이다.
* 이 영역에서 발생하는 GC 작업을 `Major GC`라고 한다.

> `Major GC` : Minor GC에 비해 비교적 적게 발생하며, STW가 발생한다. 모든 객체를 확인해야 하기 때문에 느리고, 이전 Generation의 영향을 받는다. 성능을 위해 메이저 GC의 빈도를 최소화해야 한다.

### permanent generation

* 클래스와 메서드 등의 메타데이터가 저장되는 영역 (Full GC 시에 포함되는 영역이다.)
* 런타임에 앱에서 사용 중인 클래스를 기반으로 저장되며, SE 라이브러리도 포함될 수 있다.
* 새로운 데이터를 저장할 공간이 필요한 경우 더이상 필요하지 않게된 데이터를 수집/언로딩 할 수 있다.

## GC Process 과정

1. 새로 생성된 모든 객체가 young generation의 eden 영역에 할당된다.
2. eden 영역이 가득차면 마이너 GC가 eden 영역의 객체를 수집한다.
3. 첫번째 마이너 GC 작업을 진행한다. 참조 객체들은 첫번째 survivor("from" survivor space) 영역으로 이동시키고, 미참조 객체들은 eden 영역이 비워질 때 제거한다.
4. 두번째 마이너 GC 작업을 진행한다.  참조 객체와 첫번째 survivor 영역의 객체들은 두번째 survivor 영역으로 이동한다. 작업 완료되면 eden과 from survivor 영역이 모두 비워진다.
5. 3\~4번 과정을 반복하며 교대로 survivor 영역을 바꿔가며 객체를 이동시킨다. 객체를 이동시키면서 age가 늘어난다.
6. age가 특정 임계값을 넘긴 객체는 old generation 영역으로 이동한다.
7. 최종적으로 old generation 영역에서 메이저 GC가 발생한다.&#x20;

<figure><img src="../../../.gitbook/assets/image (22).png" alt="" width="375"><figcaption><p>새로운 객체가 eden 영역에 할당</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3) (1) (2).png" alt="" width="375"><figcaption><p>참조중인 객체만 survivor 영역으로 이동</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption><p>마이너 GC 작업을 계속 진행하며 참조 객체의 age 증가</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (16).png" alt="" width="375"><figcaption><p>특정 age를 넘긴 객체는 old generation(Tenured)로 이동</p></figcaption></figure>

## GC Roots

### 개념

* GC를 위한 특별한 객체로 GC 프로세스의 시작점이다.
* GC 루트로부터 직/간접적으로 참조되는 객체는 GC 대상에서 제외된다.
* GC 알고리즘의 대부분(Hotspot JVM)은 GC 루트부터 객체 그래프를 순회하기 시작하여 참조되고 있는 객체들을 식별하고 GC 수집 대상을 찾는 형태이다.
* GC 프로세스는 애플리케이션에 정의된 모든 GC 루트에 대해 실행된다.

### 타입

* GC 루트의 타입으로는 여러가지가 존재한다.
* 클래스
  * 클래스 로더에 의해 로딩된 클래스
  * 해당 클래스의 스태틱 필드의 참조도 포함된다.
* JVM 스택의 LVA(Local Variable Array)
  * LVA의 지역 변수, 매개 변수
* 활성화 상태의 스레드
* JNI 참조 
  * JNI 호출을 위해 생성된 네이티브 코드 Java 객체 
  * 지역 변수, 매개 변수, 전역 JNI 참조 등
* 동기화를 위해 모니터로 사용되는 객체 
  * ex) `synchronized` 블록에서 사용(참조)되는 객체
* GC 루트 용도로 JVM 정의/구현한 GC 처리되지 않는 객체 
  * ex) Exception 클래스, system(custom) 클래스 로더 등

## GC Reference Counting

* 참조 객체의 참조 횟수를 세어 GC를 처리하는 방법이 있었지만 현재는 사용되지 않는 방법이다.
* 구현이 간단하고 횟수가 0이 되었을 때 즉시 제거할 수 있어, GC가 처리되어야 할 때마다 발생하는 STW 등을 피할 수 있다.
* 하지만 순환 참조 문제를 해결하기 어렵고 카운팅을 위한 추가 메모리가 필요하였기에 사용하지 않게 되었다.

#### 참조

[https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
