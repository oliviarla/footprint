# Docker Swarm

## 특징

* 컨테이너 오케스트레이션 툴으로, 서로 다른 각각의 호스트에 어느 컨테이너를 배치할 것이고 어떻게 통신을 제어할 것인지 관리할 수 있다.
* 컨테이너 스케줄링 및 상태 관리를 해준다.
* 클러스터의 쉬운 설치 및 빠른 구성이 가능하다.
* 낮은 러닝커브와 심플한 클러스터 구조로 관리자의 운영 부담이 적다.

## 용어 정리

#### 노드

* 클러스터를 구성하는 개별 도커 서버
* 매니저 노드
  * 클러스터 관리와 컨테이너 오케스트레이션을 담당
  * K8S의 Master Node와 같은 역할을 한다.
  * 클러스터의 모든 변경 사항은 리더 노드를 통해 전파되며, 나머지 노드들은 리더 노드와 동기화된 상태를 유지한다.
* 워커 노드
  * 컨테이너 기반 서비스들이 실제로 구동되는 노드
  * 매니저 노드도 기본적으로 워커 노드의 역할을 수행할 수 있다.

#### Stack

* 하나 이상의 서비스로 구성된 다중 컨테이너 애플리케이션 묶음
* Docker Compose와 유사한 형태의 yml 파일로 배포할 수 있다.

#### Service

* 워커 노드에서 수행하고자 하는 작업을 정의해놓은 것으로 기본적인 어플리케이션 생성 단위이다.
* 하나의 서비스는 하나의 이미지를 기반으로 구동된다.

#### Task

* 도커 컨테이너를 구성하여 노드에 서비스를 구동하는 단위
* 지정된 replica 개수에 따라 한 서비스에 대해 여러 태스크가 존재할 수 있다.

<figure><img src="../../.gitbook/assets/image (170).png" alt=""><figcaption></figcaption></figure>

## 사용법

#### 노드 구성

* 매니저 노드로 둘 호스트에서 아래 명령어를 수행한다.

```java
docker swarm init
```

```java
[root@ncp-2c4-001 ~]# docker swarm init
Swarm initialized: current node (vl7o9n8odml1blmyhk9cfrjvd) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token <token> <address>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

* 위 명령어 수행을 통해 얻은 docker swarm join 명령어를 복사한 후, 워커 노드로 둘 호스트에서 아래 명령어를 수행하면 노드가 등록된다.

```java
docker swarm join --token <token> <address>
```

* 아래 명령어로 노드를 확인할 수 있다.

```java
docker node ls
```

```java
ID                            HOSTNAME      STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
vl7o9n8odml1blmyhk9cfrjvd *   ncp-2c4-001   Ready     Active         Leader           24.0.7
vtm8iho7yedmx6sisctngkro7     ncp-2c4-002   Ready     Active                          1.13.1
qgdiyu46ugz6u58hghjryokkb     ncp-2c4-003   Ready     Active                          1.13.1
```

## K8S와 비교

<figure><img src="../../.gitbook/assets/스크린샷 2024-01-04 오후 2.40.41.png" alt=""><figcaption></figcaption></figure>

* Scalibility
  * Docker Swarm은 사용자 요청에 의한 수동 확장만 가능하다.
  * K8s는 Horizontal Pod Autoscaler를 통해 트래픽 기반 자동 확장 가능
* High Availability
  * Docker Swarm은 매니저 노드의 관리를 통해 가용성을 확보할 수 있다. 한 Worker Node의 replica에 장애가 발생하면 다른 Worker Node에 replica를 구성하여 재스케줄링한다.
* Network
  * 클라이언트와의 통신에서 네트워크 훅을 최소화하기 위해 host network를 사용할 수 있다.
