# 기본 개념

## Container Orchestration

* 보통의 프로덕션 환경이라면 다수의 컨테이너를 실행시킬 것이다. 이렇게 실행중인 다수의 컨테이너들을 관리하는 기술이 필요하다.
* 즉, 장애가 발생하면 해당 프로세스를 재실행해주고 부하가 늘어나면 컨테이너를 늘리는 등의 작업을 맡아주는 기술이 필요하다.
* Docker Swarm, MESOS, Kubernetes는 Container Orchestration 툴이다.

## 쿠버네티스 구성요소

* 노드
  * 쿠버네티스가 설치되어 있는 물리 혹은 가상의 장비
  * 컨테이너가 실행될 수 있는 장비를 의미한다.
* 클러스터
  * 노드들의 집합
  * 여러 노드를 둠으로써 부하를 분산할 수 있으며, 하나의 노드에 장애가 발생해 사용할 수 없는 경우 다른 노드를 사용할 수 있어 고가용성에 유리하다.
* 마스터
  * 클러스터를 관리하는 노드
  * 마스터 노드는 일반 노드와 달리 컨테이너가 실행되지 않는다.
  * 클러스터에 속하는 노드의 정보를 가지며, 노드들을 모니터링하며 실패할 경우 워크로드를 이동하는 등의 역할을 한다.
* 컴포넌트
  * 쿠버네티스를 설치하면 아래의 컴포넌트들이 설치된다.
  * API Server
    * CLI를 제공해 사용자, 장비 관리 등 쿠버네티스 클러스터와 상호작용할 수 있도록 한다.
  * etcd
    * 분산형 key value 저장소로 클러스터 관리를 위한 정보들을 저장한다.
    * 마스터들 간에 충돌이 없도록 락을 잘 구현해야 한다.
  * Scheduler
    * 여러 노드에 작업 또는 컨테이너를 분산하는 역할을 한다.
    * 새로 생성된 컨테이너를 확인하고 노드에 할당한다.
  * Controller
    * 노드, 컨테이너 또는 엔드포인트가 다운될 때 이를 감지하고 대응하는 역할을 한다.
  * Container Runtime
    * 컨테이너를 실행시킬 때 사용되는 소프트웨어
    * Docker를 비롯하여 Rocket, Cryo 등의 컨테이너 툴을 사용할 수 있다.
  * Kubelet
    * 클러스터 내의 각 노드에 실행되는 에이전트
    * 컨테이너가 노드에서 잘 수행되고 있는지 확인하고 마스터 노드와 통신하는 역할을 한다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>마스터와 워커 노드의 구성도</p></figcaption></figure>
