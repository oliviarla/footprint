# 카프카 개요

* 링크드인에서는 파편화된 데이터 수집 및 분배 아키텍처를 운영하는 데에 어려움을 겪었다.
* 대규모 아키텍쳐에서 데이터를 생성하는 소스 애플리케이션들과 데이터를 최종 적재하는 타겟 애플리케이션들을 직접 연결하는 것은 매우 복잡하기 때문이다.
* 소스 애플리케이션과 타겟 애플리케이션을 연결하는 파이프라인 개수가 많아지면서 소스 코드, 버전 관리에 이슈가 생기고 장애가 그대로 전파되는 등의 문제가 발생했다.
* 아래는 카프카를 도입하기 전 링크드인의 아키텍처로, 직접 애플리케이션끼리 연결해 데이터를 처리하였기 때문에 데이터 파이프라인이 복잡하였다.

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* 아래는 카프카를 도입한 후 링크드인의 아키텍처로, 중앙집중화된 형태로 데이터를 한 곳에 모아 처리하고 관리할 수 있게 되었다. 또한 소스 애플리케이션과 타겟 애플리케이션의 의존도가 최소화되어, 소스 애플리케이션은 타겟 애플리케이션의 상태에 관계 없이 카프카로 데이터를 입력하면 된다.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>



* 카프카는 직렬화/역직렬화를 통해 Byte Array 형태로 통신하므로 데이터 포맷의 제약이 없다.
* 상용 환경에서 카프카는 최소 3대 이상의 서버(브로커)에서 분산 운영하고, 프로듀서로부터 전송받은 데이터를 파일 시스템에 기록한다.
* 카프카 클러스터 중 일부 서버에 장애가 발생해도 데이터 복제가 지속적으로 이뤄지므로 안전한 운영이 가능하다.
* 데이터를 묶음 단위로 전송하는 배치 전송 기능을 제공하여 낮은 지연과 높은 데이터 처리량을 달성할 수 있다.

## 빅데이터와 카프카

* IT 서비스에서는 디지털 정보로 기록되는 것들을 모두 저장해두며 실시간으로 저장되는 데이터의 양은 TB\~EB 단위로 어마어마하게 많다.
* 이러한 빅데이터로 적재되는 데이터의 타입은 아래와 같이 나뉜다.
  * 정형 데이터: 기존 RDBMS에서 사용 가능한 스키마 기반의 데이터 타입
  * 비정형 데이터: 일정 규격이나 형태를 지니지 않은 데이터 타입으로, 그림/영상/음성 등의 데이터도 이에 포함된다.
* 데이터 레이크란 빅데이터를 저장하고 활용하기 위해 데이터를 모으는 저장공간이다. 데이터 웨어하우스와 달리 필터링되거나 패키지화되지 않은 데이터가 저장된다.
* 서비스에서 발생한 데이터를 데이터 레이크로 모으려면 단순히 end-to-end 방식으로 넣는 것보단 데이터를 추출하고 변경, 적재하는 과정을 엮은 데이터 파이프라인을 구축해야 한다.
* 카프카는 이러한 데이터 파이프라인을 구축할 때 사용하기에 적합하도록 설계되었다.

## 카프카의 장점

### 높은 처리량

* 카프카는 데이터를 주고받을 때 묶어서 전송하여 네트워크 통신 횟수를 최소화한다.
* 많은 양의 데이터 묶음을 배치를 통해 빠르게처리하여 대용량의 실시간 로그 데이터를 처리하는 데에 적합하다.
* 또한 파티션 단위로 동일 목적의 데이터를 분배하고 병렬 처리할 수 있다. 파티션만큼 컨슈머 개수를 늘리면 시간 당 데이터 처리량을 늘릴 수 있다.

### 확장성

* 데이터의 양에 따라 카프카 클러스터의 브로커 개수를 무중단으로 스케일 인-아웃 할 수 있다. 따라서 확장성 있고 안정적인 운영이 가능하다.

### 영속성

* 데이터를 메모리에 저장하지 않고 파일 시스템에 적재한다.
* 운영체제 레벨에서 파일 I/O 성능 향상을 위해 페이지 캐시 영역에 메모리를 사용하여 한 번 읽은 파일 내용을 저장해둔다. 이를 통해 애플리케이션 장애가 발생하더라도 안전하게 데이터를 다시 처리할 수 있다.

### 고가용성

* 카프카 클러스터는 데이터의 복제를 통해 고가용성을 보장한다. 프로듀서로부터 전달받은 데이터를 여러 브로커에 저장하여 하나의 브로커에 장애가 발생하더라도 복제된 데이터가 다른 브로커에 있으므로 문제가 발생하지 않는다.
* 카프카 클러스터를 구축할 때 브로커 개수의 제한은 없지만 1대로 운영하면 브로커가 장애나면 바로 서비스가 장애가 날 것이므로 테스트 시에만 사용해야 하고, 2대로 운영하면 브로커간 복제 시간 차이로 인해 일부 데이터의 유실 가능성이 있다.
* 데이터 유실을 막기 위해 min.insync.replicas 옵션에 복제가 반드시 완료되어야 하는 최소 브로커 개수를 설정할 수 있다. 이 옵션 값보다 브로커 개수가 적을 경우 토픽에 데이터를 더이상 넣을 수 없다.&#x20;
* 상용에서 카프카 운영 시에는 3대 이상의 브로커로 클러스터를 구축해야 데이터 유실을 방지하고 장애가 나지 않도록 할 수 있다.

## 데이터 레이크 아키텍처와 카프카

* 데이터 레이크 아키텍처는 두 종류가 있다.
* 람다 아키텍처
  * 초기 빅데이터 플랫폼에서 end-to-end 방식으로 데이터를 모으는 구조에는 아래와 같은 단점이 있었다. 람다 아키텍처는 이러한 단점을 개선하기 위해 생겨났다.
    * 유연한 아키텍쳐 변경이 어렵고 실시간으로 생성되는 데이터에 대한 인사이트를 서비스 애플리케이션에 빠르게 전달하지 못한다.
    * 소스 애플리케이션으로부터 생성된 데이터의 히스토리 파악이 어렵고 데이터 가공으로 인해 데이터가 파편화되어 데이터 거버넌스(데이터 표준 및 정책)를 지키기 어려워졌다.
  * 람다 아키텍처는 데이터 레이크를 3가지 레이어로 나누어 각각의 역할을 분리한다.
    * 배치 레이어: 배치 데이터를 모아 일정 주기마다 일괄 처리한다.
    * 서빙(serving) 레이어: 가공된 데이터를 서비스 애플리케이션에서 사용할 수 있도록 저장한다.
    * 스피드(speed) 레이어: 서비스에서 생성된 데이터를 실시간으로 분석한다. 분석된 데이터는 서비스 애플리케이션에서 바로 사용할 수도 있고 서빙 레이어로 보낼 수도 있다. 카프카는 실시간 데이터를 짧은 지연시간으로 처리할 수 있고, 카프카 스트림즈 등의 스트림 프로세싱 도구를 사용해 실시간 데이터 분석이 가능하여 스피드 레이어에서 사용될 수 있다.
  * 데이터를 배치 처리하는 레이어와 실시간 처리하는 레이어가 나뉘기 때문에, 배치 데이터와 실시간 데이터를 융합해 처리하려면 유연하지 못한 파이프라인을 생성해야 하는 등 각 레이어 구현체의 파편화가 발생한다.
* 카파 아키텍처
  * 배치 레이어를 두지 않고 모든 데이터가 스피드 레이어를 통해 처리되도록 한다.
  * 스피드 레이어에서는 서비스 애플리케이션으로부터 생성되는 모든 종류의 데이터를 처리해야 한다.
  * 배치 데이터는 특정 기간 내에서 발생한 한정된(bounded) 데이터를 의미하며, 스트림 데이터는 한정되지 않아(unbounded) 시작 데이터와 마지막 데이터가 명확히 정해지지 않은 데이터를 의미한다.
  * 배치 데이터를 스트림 프로세스로 처리하기 위해서 모든 데이터를 로그(데이터의 집합)으로 바라보고 일정한 번호를 붙인다. 각 시점의 스냅샷 배치 데이터를 사용하는 대신, 각 시점의 배치 데이터 변경 기록을 시간 순으로 나타내어 배치 데이터를 표현할 수 있다.
  * 로그로 배치 데이터와 스트림 데이터를 저장하고 사용하기 위해서는 변경 기록이 일정 기간동안 삭제되면 안되고 지속적으로 추가되어야 한다.
  * 모든 데이터가 스피드 레이어를 거치므로, 이를 구성하는 데이터 플랫폼은 반드시 고가용성과 장애 허용(fault tolerant) 특징을 가져야 한다. 카프카는 이러한 특성을 가지고 있어 스피드 레이어에서 사용되기에 적합하다.