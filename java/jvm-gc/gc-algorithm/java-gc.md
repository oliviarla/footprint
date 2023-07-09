# Java GC 알고리즘

GC 종류

* Epsilon GC - 다른 GC와 결이 다르다 (메모리 수집 X)
* GC Phase properties
  * a parallel phase
    * 멀티 스레드로 실행
  * a stop the world phase
    * 앱이 멈추는 방식
  * a concurrent phase
    * 백그라운드에서 실행되어 앱 작업과 동시에 실행

#### Serial GC

#### Parallel GC

* 멀티스레드를 활용하는 GC
* serial GC 보다 STW 시간을 단축하는 개념

#### CMS GC

* 단계를 나누어 앱실행과 함께 할 수 있는 작업은 함께 하고, 꼭 필요한 경우에만 STW phase 사용
* Initial Mark
* Concurrent Mark
  * 앱실행과 동시에 마킹 처리
* Remark
* Concurrent sweep
  * 앱실행과 동시에 sweep 처리
* Full GC
  * Old Generation의 점유율이 기준치에 도달하면 CMS 시작됨
  * GC root를 찾는 등에 의해 Initial Mark는 STW가 필요하다
  * compaction 작업이 없어 메모리 파편화 발생

#### G1 GC

* region 이라는 영역을 나눠 관리

#### Shenandoah GC

* 앱을 중단하지 않고 compaction 진행
* 실행 단계에서 참조를 변경하기 때문에 브룩스 pointer를 두어 메모리 위치를 이동시켜도 참조가 유지되도록 한다.
* STW 시간을 최소화하는 것을 목표로 한다
* 짧은 STW 시간을 위해 메모리 사용을 최대한 활용
* G1에 비해 앱의 실행이 중단되지 않도록 설계됨

#### ZGC

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

#### Epsilon GC

* GC로 인해 리소스가 뺏기는 것을 피하면서 테스팅하고 싶을 때 사용
* 스카우터?
