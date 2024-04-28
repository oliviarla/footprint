# Heap 영역을 제외한 GC 처리 영역

## 네이티브 메모리 영역

* 네이티브 메모리 영역에는 아래와 같은 데이터들이 저장된다.
  * Metaspace
  * Threads
  * Code Cache
  * GC
  * Symbols
  * Native Byte Buffers
* 이중에서 Metaspace와 Code Cache 영역은 GC가 관리해준다.

## Metaspace

* JDK 8에서 도입된 영역으로, Hotspot VM의 네이티브 메모리 관리자
* 클래스가 로딩될 때 할당되는 메타데이터에 대한 메모리 관리를 담당 
* 일반적으로 한 클래스의 메타데이터는 로딩을 담당하는 클래스 로더의 생명주기와 관련이 있다. 만약 해당 클래스 로더가 GC에 의해 제거되면 관련된 클래스들의 대량 메타데이터가 제거(소실)된다.
* GC는 Metaspace 사용량이 최대치에 도달하면 참조되지 않는 클래스의 수집을 처리(트리거)한다. 따라서&#x20;
* Permanent
  * 특수한 힙 영역이었지만, JDK 8 이후부터 Metaspace로 인해 대체되었다.
  * static 메서드와 필드에 대한 참조, primitive type 변수, 바이트코드에 대한 데이터와 이름, JIT 컴파일에 대한 정보를 포함하는 영역이었다.

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

## Code Cache

* JIT 컴파일러가 가장 많이 사용하는 영역으로, Java 바이트코드를 컴파일한 네이티브 코드를 저장해두는 곳이다.
* JVM의 `Code Cache sweeper` 스레드가 더이상 사용되지 않는 코드 세그먼트를 제거해가며 메모리를 관리한다.
* 고정된 크기로 프로그램이 실행되기 때문에 확장이 불가능하며, 용량이 가득차면 JIT 컴파일러가 동작하지 않아 성능 부하를 야기할 수 있다.
