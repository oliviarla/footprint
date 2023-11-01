# 레지스터

* 보통 현재 계산을 수행중인 값을 저장하는 데 사용된다.
* 대부분의 현대 프로세서는 메인 메모리에서 데이터를 레지스터로 옮겨와 처리한 후 해당 내용을 다시 메인 메모리에 저장하는 방식을 사용한다.
* CPU내의 메모리 계층 최상위에 위치하며 가장 빠른 속도로 접근 가능한 메모리이다.

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption><p><a href="https://leechangyo.github.io/cs/2020/05/19/20.-what-are-CPU-Registers/">https://leechangyo.github.io/cs/2020/05/19/20.-what-are-CPU-Registers/</a></p></figcaption></figure>

## 32 비트 컴퓨터가 RAM을 최대 4GB까지 밖에 사용하지 못하는 이유

* 32bit 컴퓨터는 레지스터의 크기를 32bit로 두고 사용하는 장비이다.
* 즉, 컴퓨터가 처리하는 기본 데이터의 크기(한 번에 처리할 수 있는 데이터의 크기)가 32bit이므로 2^32 만큼을 표현할 수 있다.&#x20;
* 1 byte \* 2^32 = 4,294,967,296 bytes 내의 주소밖에 표현을 못하기 때문에 4GB를 넘게 될 경우 해당 영역을 가리키지 못한다.
* 따라서 32 bit 컴퓨터는 4GB까지밖에 사용하지 못하는 것이다.
