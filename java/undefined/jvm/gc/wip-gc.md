# (WIP) GC 알고리즘

## GC 알고리즘

### GC 알고리즘 종류

> GC 알고리즘은 Java 언어의 성능과 직결되기 때문에, Java 언어와 함께 꾸준히 개선되어오고 있다.

* Serial GC
* Parallel GC (JDK 7)
* ~~CMS GC~~ (JDK 14, not available currently)
* G1 GC (JDK 9)
* Shenandoah GC (OpenJDK only)
* ZGC
* Epsilon GC
  * 메모리 수집을 하지 않기 때문에 다른 GC와 결이 다르다.

### GC Phase properties

* a parallel phase
  * 멀티 스레드로 실행 가능
* a serial phase
  * 싱글 스레드로 실행 가능
* a stop the world phase
  * 프로그램과 동시에 실행될 수 없으므로 프로그램이 일시적으로 멈춘다.
* a concurrent phase
  * 백그라운드에서 실행되어 앱 작업과 동시에 실행될 수 있다.
* an incremental phase
  * 작업 완료 전에 종료하고 나중에 이어서 작업할 수 있다.

## Serial GC

* 싱글 스레드로 동작하는 가장 간단한 GC이다.
* GC가 실행되면 모든 스레드가 STW 되어버리므로 서버 환경에 적합하지 않다.
  * 쉽게 말하면 GC가 돌 때 프로그램 자체가 잠시 멈춰버리므로 요청을 제때 처리할 수 없게 된다!
* 클라이언트 앱 또는 일시 중지 시간에 대한 제한이 없는 경우에 선택하는 GC 방식이다.
* 다음 옵션을 주어 활성화 시킬 수 있다.
  * `java -XX:+UseSerialGC -jar <Application.java>`

## Parallel GC

* 힙 영역 관리를 위해 멀티 스레드를 활용하는 GC이다.
* `Minor GC`를 처리하는 스레드를 병렬로 처리하여 Serial GC보다 훨씬 빠르게 동작한다.
* Serial GC 보다 STW 시간을 단축할 수 있다는 장점이 있다.
* GC 수행 시에는 Serial GC와 동일하게 다른 앱 스레드도 정지되는 STW 이벤트이다.
* 다음 옵션을 주어 활성화 시킬 수 있다.
  * `java -XX:+UseParallelGC -jar <Application.java>`
*   다음과 같은 파라미터를 지정할 수 있다.

    * 최대 GC 스레드 수: `-XX:ParallelGCThreas=<Number>`
    * 최대 일시 정지 시간: `-XX:MaxGCPauseMillis=<Number>`
    * 최대 목표 처리량: `-XX:GCTimeRatio=<Number>`

    > 처리량은 GC에 소요된 시간과 GC 수집 시 외부에서 소요된 시간(응용 프로그램 시간) 측면에서 측정하게 된다. 즉, (GC 시간) / (응용 프로그램 시간) = 1 / (1+n) 이 되도록 하는 n의 값을 지정할 수 있다. 예를 들어 n을 19라고 설정하면, GC에 소요되는 시간을 5% 이내가 되도록 한다.

    * 최대 힙 크기: `-Xmx<Number>`

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption><p>Serial GC vs Parallel GC</p></figcaption></figure>

## CMS GC

* Concurrent Mark Sweep 의 줄임말로 멀티 GC 스레드를 활용하는 GC이다.
* STW로 인해 응답하지 못하는 시간이 길어지지 않도록 하는 것이 이 GC 방식의 목표이기 때문에 애플리케이션 실행을 멈추지 않고 진행 가능한 Concurrent Mark, Concurrent Sweep 작업과 STW가 꼭 필요한 Initial Mark, Remark 작업으로 단계를 나누어 STW를 줄였다.
* 하지만 다른 GC 방식보다 메모리와 CPU를 더 많이 사용하고, Compaction 단계가 기본적으로 제공되지 않는 단점이 존재하였고, 결국 G1GC에 의해 대체되어 JDK 14에서 아예 제거되었다.

<figure><img src="../../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

### 수행 과정

### Minor GC

* Live 객체는 Eden과 Survivor 영역에서 다른 Survivor 영역으로 이동하다가, 특정 임계값을 넘어서면 Old 영역으로 이동한다.

### Full GC

* Old 영역의 점유율이 기준치에 도달하면 CMS 작업이 시작된다.
* 작업은 아래와 같이 5단계로 나뉜다.
* Initial Mark (STW)
  * GC root를 찾는 작업 등에 의해 Initial Mark는 STW가 발생한다.
  * 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는 것으로 끝낸다.
  * GC 루트로부터 도달 가능한 객체를 마킹 (Old 제너레이션 내에 객체는 도달 가능한 것으로 간주됨)
  * 일반적인 마이너 GC의 STW 시간에 비해 짧다.
* Concurrent Marking
  * 다른 스레드가 실행 중인 상태와 동시에 마킹 처리를 한다.
  * Initial Mark 작업으로 살아있음을 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인한다.
* Remark (STW)
  * Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한 후 사용되지 않는 객체는 마킹해준다.
* Concurrent sweep
  * 다른 스레드가 실행 중인 상태와 동시에 객체들을 sweep(제거)하여 메모리를 확보한다.
* Resetting
  * 다음 GC 작업이 원활히 수행될 수 있도록 GC threshold 등을 초기화한다.

## G1 GC

* G1(Garbage First)은 메모리 공간이 큰 멀티 프로세스 장비에서 실행되는 프로그램을 위해 설계된 GC이다.
* CMS보다 성능이 좋고 효율적이어서 CMS를 대체하게 되었다.
* **힙 메모리 영역을 동일한 크기의 region으로 분할하여 관리**한다.

### Region

* G1 GC에서 힙은 하나의 메모리 영역이다. 즉, 다른 GC가 eden, survivor, old 영역을 가지는 것과 다르게 하나의 메모리 영역만을 갖도록 한다. 아래 사진을 보면 바로 이해할 수 있을 것이다.

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption><p>여러 영역을 가지는 CMS</p></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (4) (1).png" alt="" width="375"><figcaption><p>하나의 영역만 가지는 G1</p></figcaption></figure>

* G1은 위와 같이 하나의 메모리 영역(area)을 가지지만 이 안에서 여러 region으로 나뉜다.
* 각 영역들은 가상 메모리의 연속 범위에 할당된다.
* 힙은 약 2000개 영역으로 분할되며 영역의 크기는 JVM이 시작될 때 자동으로 결정되며 1MB\~32MB 사이이다.
* Region은 Eden, Survivor 및 Old Generation 영역으로 할당될 수 있다.
* live 객체는 다른 영역으로 복사 또는 이동된다.
* Humongous Region은 표준 영역 크기의 50% 이상인 객체가 있다면 이를 보관하는 공간이다. 인접 Region들의 집합으로 구성된다.

<figure><img src="../../../../.gitbook/assets/image (46).png" alt="" width="168"><figcaption></figcaption></figure>

### Young GC

* Young Generation (Eden, Survivor) 에서 발생하는 GC이다.
* 여러 스레드에서 병렬으로 실행되며, STW가 발생한다.
* Young Generation 메모리는 연속적이지 않은 Region의 집합으로 구성되므로, 크기를 쉽게 조정할 수 있다.
* Live 객체는 Survivor Region 사이를 이동하며 Aging Threshold가 충족되면 Old Generation Region으로 이동한다.
* Eden과 Survivor의 크기는 다음 Young GC를 위해 계산하며, 크기를 측정할 때 도움울 준다.
* 다음 이미지를 보면, 참조되지 않는 객체는 제거하고 Live 객체만 모아 새로운 Young Generation Region으로 이동하여 Young GC를 마친다.

<figure><img src="../../../../.gitbook/assets/image (44).png" alt="" width="375"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (43).png" alt="" width="375"><figcaption></figcaption></figure>

### Full GC

<figure><img src="../../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

* Initial Marking (STW)
  * Old Region의 객체에 대한 참조가 있는 Survivor Region(Root Region)이 있는지 확인하여 마킹한다.
  * live 객체 마킹은 Young GC 작업과 결합하여 수행되며, 로그에 `GC pause(young)(initialmark)`라고 표기된다.
* Root Region Scan
  * Initial Marking에서 찾은 Survivor Region에 대한 GC 대상 객체 스캔 작업을 진행한다.
  * 애플리케이션이 실행되는 동안 발생되며, Young GC가 발생할 가능성이 있기 전에 작업이 완료되어야 한다.
* Concurrent Mark
  * 애플리케이션이 실행되는 동안 전체 힙에서 live 객체를 찾는다.
  * Young GC가 발생하면 중단될 수 있다.
  * 빈 region이 발견되면 다음 단계인 Remark 단계에서 즉시 제거되며, 활성을 결정하는 정보가 계산된다.
  * Liveness 정보는 앱이 실행되는 동안 내내 계산되며, liveness를 통해 GC 수집 중 가장 좋은 리전을 선별한다.
  * CMS와 같은 스위핑 단계가 없다.
* Remark (STW)
  * 애플리케이션을 잠시 멈추고 최종적으로 GC 대상에서 제외되는 live 객체를 마킹한다.
  * CMS 수집기에서 사용된 것보다 훨씬 빠른 SATB(snapshot-at-the-beginning)라는 알고리즘을 사용한다.
    * STW가 일어난 직후의 스냅샷을 사용해 live 객체에 마킹하므로, Remark 작업 중에 죽은 객체도 live 객체로 간주된다.
  * 완전히 빈 Region은 제거하고, 모든 Region을 위해 Liveness가 계산된다.
* Cleanup (STW & Concurrent)
  * live 객체와 완전히 비어있는 region에 대해 계산한다.
  * live 객체가 가장 적은 region에 대한 미사용 객체 제거를 수행한다.
  * 비어있는 region을 초기화하고, 빈 region 정보를 Freelist 리스트에 추가해 다시 사용할 수 있도록 한다.
* Copying (STW)
  * GC 대상 Region이었지만 Cleanup 과정에서 살아남은 Region의 객체들을 새(Available/Unused) Region 에 복사하여 Compaction 작업을 수행한다.
  * 앞서 말한 살아남은 region을 수집하는 과정은 Young GC와 동시에 이뤄지며 로그에 \[GC pause (mixed)]로 표기된다. 즉, Young, Old 제너레이션에 대한 작업이 동시에 이뤄진다.
  * liveness가 가장 낮아 빨리 수집 가능한 region을 선택해 수집한다.

### **용어**

#### **Remembered Sets(RSet)**

* reference를 가진 Object들이 어느 region에 있는지 알기 위해 사용하는 자료구조
* 주어진 region에 대한 참조 객체를 추적한다. 각 region 당 하나의 RSet이 존재하며 이를 통해 Region의 병렬 및 독립된 수집을 가능하게 한다. RSet이 전체 영역에서 차지하는 비율은 5% 이내이다.
* GC 로그에서 이 Reference 정보를 갱신(update)하고 검색하는데 소요되는 시간을 확인 할 수 있다.

> _Baker's Incremental GC 알고리즘, Baker's Incremental Copying Collector 알고리즘_ 을 참고하자. Reference를 가진 Object를 찾아 라이브 객체를 복사하는 알고리즘이다.

**Collection Sets(CSet)**

* GC가 수행될 Region 집합
* CSet 내의 데이터는 GC 동안 모두 비워진다(복사되거나 이동됨).
* Region 집합은 Eden, Survivor, Old Generation으로 이루어질 수 있다.
* CSet이 JVM에서 차지하는 비율은 1% 이내이다.
* GC 로그에서 CSet을 선택하고 Processing 후 CSet을 해제하는데 걸린 시간을 확인할 수 있다.
* _Evacuation Pauses와 Mixed GC를 수행할 때 사용하는 자료구조_

## Shenandoah GC

* OpenJDK에서만 존재하는 GC
* STW 시간을 최소화하는 것을 목표로 하며, 짧은 STW 시간을 위해 메모리 사용을 최대한 활용한다.
* G1과 같이 힙 영역이 리전이라는 가상의 공간으로 나눠지지만 G1을 비롯한 GC들과 달리 힙을 제너레이션으로 나누지 않는다.
* 앱과 동시에 더 많은 GC 작업을 수행해 일시 중지 시간을 줄인 GC (CMS와 G1 처럼)
* 동시 압축 기능이 추가되어 힙 크기와 상관없이 일관된 일시 중지 시간을 가진다.
* 응답 시간이 짧고 GC STW가 예측이 가능할 경우 유효한 GC 알고리즘
* 브룩스 포인터를 통해 참조 재배치 메커니즘을 처리한다.
  * 브룩스 포인터(Brooks Pointer): 각 객체의 실제 위치를 가리키는 필드
  * 실행 단계에서 참조를 변경하기 때문에 브룩스 pointer를 두어 메모리 위치를 이동시켜도 참조가 유지되도록 한다.
* 앱을 중단하지 않고 compaction 진행
* G1에 비해 앱의 실행이 중단되지 않도록 설계됨

### 수행 과정

* Initial Mark (STW)
* Concurrent Marking
  * 앱 실행과 동시에 힙 내에 객체를 추적하는 단계
  * 이 작업의 시간은 Live 객체의 수와 객체 그래프에 따라 달라짐
  * 또한 앱은 이 단계에 새 데이터(객체)를 할당 가능 (이 작업이 앱의 실행을 중단시키지 않음)
* Final Mark (STW)
  * 잠시 중단되었던 마킹/업데이트 큐를 비우고 루트 셋을 다시 스캔하여 동시 마킹 작업을 완료하는 단계
  * 완료 후 비워져야 할 리전을 파악, 다음 단계를 위해 비우고 초기화 등 작업을 수행
  * 대기열을 비우고 루트 셋을 비우는 작업에 따라 STW 시간은 크게 달라짐
* Concurrent Cleanup
  * 즉각적인 GC 영역으로 동시 마킹 작업 후 Live 객체가 없는 리전을 회수하는 단계
* Concurrent Evacuation
  * 컬렉션 셋을 다른 리전으로 `동시에` 이동(복사)하는 단계 (이것이 다른 GC와의 주요 차이점)
  * 이 단계는 앱과 함께 다시 실행되어 여유 공간인 리전을 할당할 수 있음
  * 선택한 컬렉션 셋의 크기에 따라 작업 시간은 크게 달라짐
* Init Update Refs (STW 중 제일 짧음)
  * 이동한 객체들의 참조 업데이트를 초기화하고, 모든 GC와 앱 스레드가 제거 작업을 완료했는지 확인하며 다음 단계를 위해 GC를 준비하는 단계 (이외에는 거의 아무것도 하지 않음)
  * 즉 이동 및 제거 등 작업이 완료된지 확인한 후에 다음 단계를 위해이동한 객체에 대한 포인터 등을 업데이트 작업 등을 초기화(준비)
* Concurrent Update References
  * 힙을 살펴보고, `Concurrent Evacuation` 작업 중 이동된 객체에 대한 참조를 업데이트하는 단계
  * 다른 GC와의 주요 차이점이다.
  * 해당 작업은 앱의 실행과 동시에 진행되며 힙의 객체 수에 따라 작업 시간이 달라짐 (하지만 객체 그래프에 영향을 받지는 않음)
* Final Update Refs (STW)
  * 기존에 루트 셋을 업데이트하여 참조 업데이트 단계를 완료하는 단계
  * 컬렉션 셋에서 리전을 재활용하며 루트 셋의 크기에 따라 STW 시간은 달라짐
* Concurrent Cleanup
  * 참조가 없는 컬렉션 셋 리전을 회수

## ZGC

* 힙 크기에 영향을 받지 않는다
* 참조의 메타데이터 비트 확인, 참조 얻기 전 일부 처리 수행
* 64 bit 환경에서만 사용 가능
* 동작 과정
  1. marking
  2. Reference Coloring
     * 참조 컬러 - 4개의 메타데이터 비트 정보로 remapped 되었는지 확인 가능
  3. Relocation
  4. Remapping & Load Barriers
     * 로드 배리어를 사용해 힙에서 객체 참조를 로딩할 때마다 실행되도록 한다.
     * 로드배리어가 Remapping을 수행해준다.

## Epsilon GC

* 메모리를 할당하지만 실제로 회수하지 않는 GC이다.
* 사용 가능한 메모리를 모두 사용하면 프로그램이 종료된다.
* 메모리 풋프린트와 처리량을 낮추면서 대기시간이 가장 낮은 형태로  최소한의 작업만 수행하는 데 초점을 맞추고 있다.
* GC의 영향을 피하면서 성능이나 메모리 부하 테스팅하고 싶을 때 사용할 수 있다.
