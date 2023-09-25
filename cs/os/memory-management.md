# Memory Management



{% hint style="info" %}
CPU활용을 높이기 위해 메모리를 효율적으로 관리하는 방법을 소개합니다.
{% endhint %}

## Swap

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 메인 메모리에 원하는 데이터가 없다면 하드 디스크의 데이터 접근해 메모리로 올려두고(swap in), 메모리에서 사용하지 않을 것 같은 데이터는 하드디스크에 저장한다. (swap out)
* 어떤 프로세스를 swap in, swap out 할지는 CPU 스케줄러에 의해 결정된다.
* 하드디스크의 프로그램을 내렸다가 올렸다가 하므로 속도가 느리며 context switching 시간이 많이 걸린다.
* 하지만 이를 통해 부족한 메모리에서도 더 크고 많은 프로세스를 동시에 실행할 수 있다.
* 프로세스 단위로 진행할 경우 외부 단편화 문제 발생

## Contiguous Allocation

* 연속 메모리 할당
  * 하나의 연속된 메모리 블록에 프로세스나 파일의 데이터를 할당하는 것
  * 사용 가능한 메모리 블록이 전체 메모리 공간에 걸쳐 여기저기 랜덤하게 분포되지 않는다.

<figure><img src="../../.gitbook/assets/image (3) (2).png" alt="" width="284"><figcaption></figcaption></figure>

*   외부 단편화(External Fragmentation)

    * 메모리에서 빈 공간보다 프로세스 크기가 크면 프로세스를 할당할 수 없어 발생하는 문제
    * 프로세스보다 작은 page 단위로 메모리를 관리해 외부 단편화 문제를 해결할 수 있다.
      * 프로세스를 page 단위로 나누고, 프레임 단위로 매핑하여 메모리를 효율적으로 사용할 수 있다.
    * 아래와 같이 프로세스가 할당된 Assigned Space가 아닌 Fragment에 새로운 프로세스를 할당할 수 있으나, 프로세스가 50KB의 메모리 공간을 필요로 할 경우 할당이 불가능하다.

    <figure><img src="../../.gitbook/assets/image (2) (2).png" alt=""><figcaption></figcaption></figure>
* 내부 단편화(Internal Fragmentation)
  * 외부 단편화 해결을 위해 page 단위로 메모리를 관리하게 되었는데, 이 page 내부에 할당된 프로세스 외에 공간이 남아 발생하는 문제
  * 아래와 같이 Assigned Space에 프로세스가 Used Space 만큼만 사용해 Fragment가 사용되지 않는 것을 볼 수 있다.

<figure><img src="../../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

## TLB

* Translation Lookaside Buffer
* page table이 너무 크면 검색 시간이 오래 걸리므로 TLB에 최근 사용한 페이지를 저장한다.
* page table 대신 TLB(하드웨어)에 먼저 접근해 frame number 를 찾는다.
* 만약 TLB에서 frame number를 찾으면 physical memory에 바로 접근한다.
* TLB에서 frame number를 못찾을 때에는 page table에 접근한다.
* MM=400ms, TLB=50ms, Hit Ratio=90%일 때
  * TLB가 없을 때 총 소요 시간: page table 접근 시간+ 메인메모리 접근 시간 = 2\* 400ms=800ms
  * TLB가 있을 때 총 소요 시간: Hit rate\*(TLB 접근 시간+ 메인메모리 접근 시 간)+(1-Hit rate)(page table 접근 시간 + 메인메모리 접근 시간) = 0.9(50ms+400ms)+0.1\*(2\*400ms) = 490ms&#x20;
* paging의 단점
  * 내부 단편화 문제가 발생한다.
  * page table에 접근할 때 비용이 발생한다.
  * page table의 크기가 클수록 frame 찾을 때 많은 시간이 소요된다.
  * TLB가 paging이 가지는 오버헤드를 줄일 수 있지만 context switch가 자주 발생하면 프로세스마다 TLB가 다르기 때문에 TLB를 fresh out(교체)하는 비용 이 많이 들 수 있다.

<figure><img src="../../.gitbook/assets/image (23).png" alt="" width="375"><figcaption></figcaption></figure>



#### 참고 자료

{% embed url="https://www.geeksforgeeks.org/difference-between-internal-and-external-fragmentation/?ref=gcse" %}

{% embed url="https://www.geeksforgeeks.org/difference-between-contiguous-and-noncontiguous-memory-allocation/" %}
