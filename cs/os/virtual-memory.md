# Virtual Memory

### Virtual Memory

* 보조 메모리를 마치 메인 메모리의 일부인 것처럼 사용할 수 있도록 하는 메모리 관리 기법
* 하드웨어와 소프트웨어를 모두 사용하여 물리적 메모리 부족을 보완하도록 한다.
* RAM(Random Access Memory)에서 디스크 스토리지로 데이터를 일시적으로 전송한다.
* 메모리 청크를 디스크 파일에 매핑한다.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

* "M" 이라는 페이지를 참조하기 위해 명령어 입력 시 TLB에서 먼저 확인하고, 없으면 page table에서 데이터를 찾는다.
* 시스템을 처음 시작했을 때 valid-invalid bit는 모두 invalid이다.
* OS에서 free frame에 데이터를 올려놓았으면 invalid를 valid로 바꾸어 준다.

> free frame: 빈 프레임 공간

### Segmentation





### Paging

