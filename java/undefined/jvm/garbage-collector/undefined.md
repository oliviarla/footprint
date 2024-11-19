# 기본 동작

## GC Process

* 가장 기본적인 GC의 동작으로는 Mark, Sweep, Compact가 있다.

### **Mark**

* 참조 객체와 참조되지 않는 객체를 식별하는 프로세스
* 모든 객체를 스캔하므로 시간이 많이 걸릴 수 있다.
* 참조되지 않는 객체를 제거하기 전에 미리 수행되어야 하는 작업이다.
* 마킹 프로세스가 완료되면 아래 그림과 같이 참조되지 않는 주황색 부분만 마킹된다.

<figure><img src="../../../../.gitbook/assets/image (18) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

### **Sweep**

* 아무 곳에서도 참조되지 않아 마킹된 미사용 객체를 제거하는 프로세스
* 객체가 제거되어 새 객체를 할당할 수 있는 여유 공간에 대한 참조를 메모리 할당자가 들고 있도록 한다.

<figure><img src="../../../../.gitbook/assets/image (21).png" alt="" width="563"><figcaption></figcaption></figure>

### **Compact**

* Mark, Sweep 작업으로 인해 생긴 메모리 단편화를 없애는 프로세스
* 남아있는 참조 객체를 이동해 메모리 압축 작업을 수행하면 새 메모리 할당이 훨씬 빨라진다.
* 이 때에도 새 객체를 할당할 수 있는 여유 공간 블록에 대한 참조를 메모리 할당자가 들고 있도록 한다.

<figure><img src="../../../../.gitbook/assets/image (12) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## JVM Generations

* 많은 객체가 할당되면 compact, marking이 오래 걸려 GC 작업 시간이 증가하는 현상을 보완하기 위해 오래된 객체, 새로운 객체를 나누어 관리하는 방법이다.
* 아래 그림과 같이 힙 영역을 **객체의 오래된 정도를 기준으로** 영역을 나누어, 가장 최신의 객체는 `Young Generation`, 오래된 객체는 `Old Generation`, 프로그램이 죽을 때까지 사용되는 객체는 `Permenent Generation`에 담을 수 있다.
* 오래된 객체는 앞으로도 계속해서 참조될 확률이 높기 때문에 GC 빈도를 줄이고, 새로운 객체를 담는 영역은 비교적 자주 GC 작업을 하여 GC에 영향받는 메모리 영역을 대폭 줄일 수 있다.

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **young generation**

* 새로운 객체가 할당되는 영역이며, 이는 다시 1개의 Eden 영역과 2개의 Survivors 영역으로 나뉜다.
* `young generation`에서 오랫동안 존재한 객체는 `old generation` 으로 이동한다.
* `Minor GC`가 발생한다.

{% hint style="info" %}
**Minor GC**

eden 영역과 객체들이 있는 survival 영역을 돌며 참조되지 않는 객체를 제거하고, 비어있는 survival 영역으로 참조중인 객체를 모두 이동시키는 프로세스이다. STW가 발생한다.
{% endhint %}

{% hint style="info" %}
**STW(Stop The World) 이벤트**

작업이 완료될 때 까지 모든 애플리케이션 스레드가 중지되는 이벤트이다. 동시 처리가 안될 때 어쩔 수 없이 모든 작업을 중단해야 하기 때문이다.
{% endhint %}

#### **eden 영역**

* 인스턴스 생성 시 eden 영역에 할당 된다.
* 살아남을 객체만 마킹하는 작업을 수행한 후, survival 영역으로 이동한다.

#### **survival 영역**

* from과 to 의 두 영역으로 나뉘며, 객체들을 from에서 to로 이동시키며 GC를 실행한다.
* survival 영역에서 왔다갔다 이동하며 설정된 임계값(threshold)을 넘긴 객체들은 오래된 객체로 간주되어 Old Generation 영역으로 이동한다.

### old generation

* 오랫동안 참조된 객체가 저장되는 영역이다.
* 이 영역에서 발생하는 GC 작업을 `Major GC`라고 한다.

{% hint style="info" %}
**Major GC**

Minor GC에 비해 비교적 적게 발생하며, STW가 발생한다. 모든 객체를 확인해야 하기 때문에 느리고, 이전 Generation의 영향을 받는다. 성능을 위해 메이저 GC의 빈도를 최소화해야 한다.
{% endhint %}

### permanent generation

* 클래스와 메서드 등의 메타데이터가 저장되는 영역 (Full GC 시에 포함되는 영역이다.)
* 런타임에 앱에서 사용 중인 클래스를 기반으로 저장되며, SE 라이브러리도 포함될 수 있다.
* 새로운 데이터를 저장할 공간이 필요한 경우 더이상 필요하지 않게된 데이터를 수집/언로딩 할 수 있다.

## 기본 GC 동작 방식

* GC 방식마다 동작 방식이 다르지만 가장 기본적인 GC의 동작 방식 순서는 아래와 같이 진행된다.
* 먼저, 새로 생성된 객체는 young generation의 eden 영역에 할당된다. 기존에 생성되어 있던 객체는 survivor 영역에 존재한다.

<figure><img src="../../../../.gitbook/assets/image (22).png" alt="" width="375"><figcaption><p>새로운 객체가 eden 영역에 할당</p></figcaption></figure>

* eden 영역이 가득 차면 `minor GC`가 eden 영역과 survivor 영역에 대해 minor GC 작업을 진행한다. 현재 시점에도 여전히 참조되고 있는 객체들은 비어있는 survivor 영역(S1)으로 이동되고, 미참조 객체들은 제거한다.

<figure><img src="../../../../.gitbook/assets/image (3) (1) (2).png" alt="" width="375"><figcaption><p>참조중인 객체만 survivor 영역으로 이동</p></figcaption></figure>

* 또다시 eden 영역이 가득 차면 `minor GC` 작업을 진행한다. 참조 객체와 첫번째 survivor 영역의 객체들은 또다시 비어있는 survivor 영역(To survivor space)으로 이동한다. `minor GC` 작업이 완료되면 eden과 from survivor 영역이 모두 비워진다. 객체가 이동될 때 마다 age가 늘어나며, 이는 old generation으로 이동할 척도가 된다.

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption><p>마이너 GC 작업을 계속 진행하며 참조 객체의 age 증가</p></figcaption></figure>

* 또다시 `minor GC`를 수행할 때 age가 특정 임계값을 넘긴 객체는 old generation으로 이동한다. 이러한 과정을 반복하며 old generation 영역이 가득 차게 되면 `major GC`가 발생한다.

<figure><img src="../../../../.gitbook/assets/image (16) (1) (1).png" alt="" width="375"><figcaption><p>특정 age를 넘긴 객체는 old generation(Tenured)로 이동</p></figcaption></figure>

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
