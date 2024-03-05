# SortedSet

{% hint style="info" %}
본 문서는 Redis 7 기준으로 작성되었습니다.
{% endhint %}

## 내부 구조

* Redis의 Sorted Set은 원소 개수나 value의 크기에 따라 내부 자료구조가 달라진다.
  * 멤버 수가 128개보다 적거나 value의 길이가 64 bytes 이하라면 `Listpack`자료 구조에 저장한다.
  * 멤버 수가 128개가 넘거나 value의 길이가 65 bytes 이상이라면 `Skip List`자료 구조에 저장한다.
  * 엔트리 개수가 다시 128개가 넘게 되어 Skip List로 변환된 Sorted Set에서 엔트리 개수가 128이하로 줄었다 하더라도 `Listpack`으로 다시 바꾸지 않고, `Skip List`를 그대로 유지한다.
  *   redis 기본 설정

      ```
      zset-max-listpack-entries: 128
      zset-max-listpack-value: 64
      ```
* 참고로 List, Hashes에서도 기본적으로 Listpack를 사용하고, 원소가 늘어나거나 데이터 크기가 커지면 다른 자료구조(Linked List, Hash Table)로 변환한다.
* Sorted set은 skip list와 hash table을 모두 사용하는 dual-ported data structure 로 구현되어 있다고 한다.
* 이로 인해 원소를 추가할 때 항상 O(log(N))만큼만 소요된다.

### Zip List (Deprecated)

* 보통 자료구조에서 실 데이터 외에 메모리를 가장 많이 차지하고 있는 것은 포인터(메모리 주소)이다.
* Zip List는 포인터를 갖지 않기 위해 기존에 할당받은 메모리를 필요한 만큼 늘려서(resize) 저장하는 방식을 사용한다.
  * 삽입, 삭제 시 realloc()과 memmove()를 수행하므로 큰 데이터에 적용하려면 성능이 저하된다.
* 커널의 메모리 할당 방식에 따른 오버헤드도 줄일 수 있다.
* 데이터 구조는 아래와 같으며, 실 데이터를 제외한 zip list의 오버헤드는 11 bytes에 불과하다.
  * zlbytes: 총 바이트 수
  * zltail: 마지막 엔트리의 시작 지점의 포인터(offset)
  * zllen: 저장된 엔트리 개수
  * zlend: zip list의 끝을 나타내는 바이트 |11111111|

<figure><img src="../../../.gitbook/assets/Untitled (1) (1).png" alt=""><figcaption></figcaption></figure>

* 문제점
  * 데이터가 추가/변경될 때 prevlen이 1byte였던 엔트리의 앞에 255 이상의 길이를 가진 엔트리를 추가하는 경우, prevlen이 5bytes로 늘어나게 된다.
  * prevlen이 늘어난 엔트리의 길이가 하필 또 255 이상이 되었다면 다음 엔트리의 prevlen도 1byte에서 5bytes로 늘어날 것이다.
  * 이 현상이 연속적으로 발생하게 되면 ziplist 안의 모든 엔트리가 수정되는 현상이 불가피하다.

### Listpack

* Redis7 버전 이후로 Zip List를 대체하는 자료구조
* Listpack은 이 단점을 개선하기 위해 prevlen 필드를 제거하고 아래와 같은 데이터 구조를 사용한다.
  * tot-bytes: Listpack의 총 바이트 수
  * num-entries: 저장된 엔트리 개수

<figure><img src="../../../.gitbook/assets/Untitled 1 (1).png" alt=""><figcaption></figcaption></figure>

### Skip List

* 동시 조회 및 수정에 더 용이하다.
* 관리용 메모리 오버헤드와 리눅스 커널 메모리 할당 방식에 따른 오버헤드를 합쳐서 실제 데이터 메모리 크기의 몇 배를 사용할 수 있다.
* Sorted Set을 위한 메인 데이터 구조
* 말그대로 원소들을 n개씩 스킵하면서 탐색할 수 있는 자료구조이다.
* 포인터를 저장해두기 때문에 메모리 오버헤드가 있다.
* 레벨이 높아질수록 포인터의 빈도가 줄어든다는 의미이다.

<figure><img src="../../../.gitbook/assets/Untitled 2 (2).png" alt=""><figcaption></figcaption></figure>

* 데이터가 많을수록 Zip List에 비해 탐색 시간이 적게 걸린다.
  * 링크드 리스트에서 80이라는 수를 찾기 위해서는 13부터 시작해서 하나하나 탐색했어야 하는데, Skip List를 사용해 레벨을 높이고 포인터를 n개씩 스킵하여 생성해두면 비교하는 횟수가 적어져 탐색 속도가 빨라진다.

<figure><img src="../../../.gitbook/assets/Untitled 3.png" alt=""><figcaption></figcaption></figure>

* 각 노드의 레벨은 노드 순서나 노드 수와 관계없이 정해져야 하고, 한 번 정해지면 바뀌지 않아야 한다. 즉 2의 배수인 노드의 경우 레벨2가 되고 4의 배수인 경우 레벨4가 되면 안된다.
* 노드의 레벨은 내부적으로 확률에 의해 정해지며, 낮은 레벨이 될 확률이 높다.
* 노드의 레벨에 규칙성이 없어 노드가 몇번째인지 알기 어려우므로, span이라는 필드를 두어 순서를 알 수 있도록 한다. 파란색 사각형에 있는 수를 합쳐 value가 20인 노드의 위치가 4번째임을 알 수 있다.

<figure><img src="../../../.gitbook/assets/Untitled 4.png" alt=""><figcaption></figcaption></figure>

**출처**

[https://redis.io/docs/data-types/sorted-sets/](https://redis.io/docs/data-types/sorted-sets/)\
[http://redisgate.kr](http://redisgate.kr/redis/configuration/internal\_listpack.php)
