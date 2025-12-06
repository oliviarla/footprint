# Data Migration

## Disaster Recovery

* 온프레미스 간 재해 복구
  * 캘리포니아와 시애틀처럼 각 지역에 데이터 센터를 두고 사용하는 전형적인 재해 복구 유형으로 비용이 아주 많이 든다.
* 하이브리드 복구
  * 온프레미스를 기본 데이터 센터로 두고 재해 발생 시 클라우드를 사용하는 방식이다.
* 완전 클라우드 복구
  * AWS Cloud 내부의 여러 리전을 두어 재해 복구를 수행할 수 있다.
*   RPO vs RTO

    <figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

    * RTO
      * Recovery Point Objective
      * 복구 시점 목표를 의미한다.
      * 재해가 일어나면 RPO와 재해 발생 시점 사이에 데이터 손실이 발생한다.
      * 얼마나 자주 백업을 실행할지, 시간상 어느 정도 과거로 되돌릴 수 있는지를 결정한다.
      * 재해 발생 시 데이터 손실을 얼마만큼 감수할지 설정하게 된다.
    * RTO
      * Recovery Time Objective
      * 복구 시간 목표를 의미한다.
      * 재해 발생 시점과 RTO의 시간 차는 애플리케이션 다운타임이다.
    * 두 지표 모두 시간 간격이 짧을수록 비용은 높아진다.

### 재해 복구 전략

#### 백업 및 복구

* RPO와 RTO가 크다.
* 백업 저장 비용만 필요하므로 저렴하다.
* 예를 들어 기업 데이터 센터와 AWS Cloud 및 S3 버킷이 있을 때 시간에 따라 데이터를 백업하는 경우
  * AWS Storage Gateway를 사용할 수 있다.
  * 수명 주기 정책을 만들어 비용 최적화 목적으로 Glacier에 데이터를 입력하거나 AWS Snowball을 사용해 일주일에 한 번씩 많은 양의 데이터를 Glacier로 전송할 수도 있다.
  * Snowball을 사용하면 RPO는 대략 일주일이 되므로 재해가 발생하면 최대 일주일 치 데이터를 잃게 된다.
* AWS의 EBS, Redshift, RDS를 사용하는 경우 정기적으로 스냅샷을 만드는 간격에 따라 RPO가 달라진다.
  * 재해가 발생하면 모든 데이터를 복구해야 하므로 AMI를 사용해서 EC2 인스턴스를 다시 만들고 애플리케이션을 스핀 업하거나 스냅샷에서 Amazon RDS 데이터베이스 EBS 볼륨, Redshift 등을 바로 복원 및 재생산할 수 있다.

#### 파일럿 라이트

* 애플리케이션 축소 버전이 클라우드에서 항상 실행되고 있는 상태를 유지한다.
* 크리티컬 시스템에 대한 보조 시스템이 작동하고 있으므로 RPO가 짧고 RTO가 빠르다.
* 예를 들어 서버와 데이터베이스를 갖춘 데이터 센터가 있을 때 크리티컬 데이터베이스에서 RDS로 데이터를 계속 복제한다면 언제든 실행할 수 있는 RDS 데이터베이스를 확보하게 된다. 재해가 발생할 경우 Route 53가 데이터 센터 서버에 장애 조치를 허용해 EC2 인스턴스를 실행하면 이미 준비된 RDS 데이터베이스를 사용해 복구할 수 있다.

#### 웜(Warm) 대기

* 시스템 전체를 실행하되 최소한의 규모로 가동해서 대기하는 방법이다.
* RPO와 RTO가 줄어든다.
* 재해가 발생하면 프로덕션 로드로 확장할 수 있다.
* 예를 들어 아래와 같이 기업 데이터 센터에 역방향 프록시와 애플리케이션 서버, 마스터 데이터베이스를 포함하고, AWS Cloud는 ELB, EC2 ASG, RDB Slave를 포함하도록 구성할 수 있다. Route 53는 DNS를 기업 데이터 센터로 가리키고 재해가 발생하면 ELB를 가리키도록 장애 조치를 할 수 있다.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

#### 핫 사이트 혹은 다중 사이트 접근

* RTO가 낮지만 비용이 많이 든다.
* AWS와 온프레미스에서 active-active로 구성할 수도 있고 AWS 멀티 리전에서 active-active로 구성할 수도 있다.
* 동시에 장애 조치 할 준비가 되어있으므로 다중 DC 유형 인프라를 실행할 수 있어 유용하다.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### 재해 복구 팁

#### 백업

* EBS 스냅샷, RDS로 자동화된 스냅샷과 백업 등을 사용한다.
* S3, S3 IA, Glacier 등에 스냅샷을 규칙적으로 푸시할 수 있다.
* 수명 주기 정책, 리전 간 복제 기능을 활용해 백업할 수 있다.
* 온프레미스에서 클라우드로 데이터 공유 시 Snowball과 Storage Gateway가 유용하다.

#### 고가용성

* Route 53를 사용해 재해 발생 시 DNS를 다른 리전으로 옮긴다.
* RDS 다중 AZ, 일래스티 캐시 다중 AZ, EFS, S3 을 통해 다중 AZ 환경을 구축할 수 있다.
* 기업 데이터 센터에서 AWS로 연결 시 Direct Connect를 실행할 때 Site to Site VPN을 네트워크 복구 옵션으로 사용할 수 있다.

#### 복제

* RDS 리전 간 복제, 오로라 글로벌 데이터베이스로 복제가 가능하다.
* 온프레미스 데이터베이스를 RDS로 복제할 때 데이터베이스 복제 소프트웨어를 쓸 수 있다.
* Storage Gateway를 사용할 수 있다.

#### 자동화

* CloudFormation과 Elastic Beanstalk가 클라우드에 새로운 환경을 빠르게 재생산할 수 있도록 한다.
* CloudWatch 사용 시 CloudWatch 실패 알람이 오면 EC2 인스턴스를 복구하거나 다시 시작할 수 있다.
* AWS Lambda 역시 사용자 맞춤 자동화에 유용하다.

#### 카오스 테스트

* 재해를 직접 만들어 테스트하는 방식이다.
* Simian Army를 만들어 실제 프로덕션 환경의 EC2 인스턴스를 무작위로 종료하는 등 다양한 실패 상황을 만들어낸다. 이를 통해 장애가 발생해도 항상 잘 대처할 수 있도록 한다.

## DMS

* 데이터베이스를 온프레미스에서 AWS로 마이그레이션하게 해주는 빠르고 안전한 데이터베이스 서비스
* 복원력이 있고 자가 치유가 가능하다.
* 소스 데이터베이스를 그대로 사용할 수 있으며, 다양한 유형의 엔진을 지원한다.
* 동종 마이그레이션, 이종 마이그레이션이 지원된다.
* CDC(Change Data Capture)를 이용한 지속적 데이터 복제가 가능하다.
* EC2 인스턴스를 만들어 복제 작업을 수행하도록 해야 한다.
  * 온프레미스의 소스 데이터베이스를 준비하고 DMS 소프트웨어를 가진 EC2 인스턴스를 실행하면, 소스 데이터베이스에서 데이터를 가져와 타깃 데이터베이스에 넣는다.
* 다음 소스 데이터베이스를 지원한다.
  * 온프레미스/EC2 인스턴스에 구동된 Oracle, Microsoft SQL Server, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2
  * Azure SQL Database
  * Amazon RDS, Aurora
  * Amazon S3
  * DocumentDB
* 다음 타깃 데이터베이스를 지원한다.
  * 온프레미스/EC2 인스턴스에 구동된 Oracle, Microsoft SQL Server, MySQL, MariaDB, PostgreSQL, SAP
  * Amazon RDS
  * Redshift, DynamoDB, Amazon S3
  * OpenSearch Service
  * Kinesis Data Streams
  * Apache Kafka
  * DocumentDB, Amazon Neptune
  * Redis, Babelfish
* 소스 데이터베이스와 타깃 데이터베이스의 엔진이 다른 경우 AWS SCT(Schema Conversion Tool)을 통해 그게 데이터베이스 스키마를 엔진 간에 변환해야 한다.
* 예를 들어 OLTP를 사용한다면 SQL Server나 Oracle에서 MySQL, PostgreSQL, Aurora로 마이그레이션할 수 있다.
* 지속적 복제
  * 기업 데이터센터에 소스 데이터베이스가 있고, private subnet 상의 AWS RDS가 타겟 데이터베이스일 때 지속적 복제는 다음과 같이 진행된다.
  * AWS SCT를 서버에 설치해 스키마가 변환될 수 있도록 한다.
  *   지속적 복제를 위한 DMS 복제 인스턴스를 두어 Full load와 Change Data Capture CDC를 진행한다.

      <figure><img src="../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>


* 멀티 AZ 배포
  * 한 AZ에 DMS 복제 인스턴스를 두고, 또다른 AZ에 동기 복제를 하여 Standby Replica 상태로 둔다.
  * &#x20;하나의 AZ에서 장애가 발생했을 때 회복력을 가질 수 있고, Data Redundancy를 갖게 되어서 I/O freezes 현상을 없애고 latency spikes를 최소화할 수 있다.

## RDS and Aurora Migration

### MySQL

* RDS MySQL 데이터베이스를 Aurora MySQL로 옮겨야 하는 경우
  * 방법 1) RDS MySQL 데이터베이스의 스냅샷을 생성해서 스냅샷을 MySQL Aurora 데이터베이스에서 복원할 수 있다.&#x20;
    * 가동을 중지한 뒤 Aurora로 마이그레이션해야 하므로 일시적으로 중지된다.
  * 방법 2) RDS MySQL로부터 Amazon Aurora 읽기 전용 복제본을 생성하여, 복제본의 지연이 0이 되면 Aurora 복제본이 MySQL과 완전히 일치하므로 데이터베이스 클러스터로 승격시킨다.
    * 데이터베이스 스냅샷보다는 시간이 많이 걸리고 복제본 생성과 관련한 네트워크 비용도 발생할 수 있다.
* 외부 MySQL 데이터베이스를 Aurora MySQL로 옮겨야 하는 경우
  * 방법 1) Percona XtraBackup 기능을 사용하여 백업 파일을 생성해 Amazon S3에 두면 Amazon Aurora의 기능을 사용해서 새로운 Aurora MySQL DB 클러스터로 백업 파일을 가져올 수 있다.
  * 방법 2) MySQL 데이터베이스에서 mysqldump 기능을 실행하여 기존 Amazon Aurora 데이터베이스로 출력값을 내보낸다. 시간이 많이 들고 Amazon S3를 사용하지 않는다.
* Amazon DMS를 이용하면 MySQL 데이터베이스와 Aurora MySQL이 모두 가동된 채로 데이터베이스 간 지속적인 복제를 진행할 수 있다.

<figure><img src="../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

### PostgreSQL

* RDS PostgreSQL 데이터베이스를 Aurora PostgreSQL로 옮겨야 하는 경우
  * 방법 1) 스냅샷을 생성해 Amazon Aurora 데이터베이스에서 복원한다.&#x20;
    * 가동을 중지한 뒤 Aurora로 마이그레이션해야 하므로 일시적으로 중지된다.
  * 방법 2)  RDS PostgreSQL로부터 Amazon Aurora의 읽기 전용 복제본을 생성하여, 복제 지연이 0이 될 때까지 기다렸다가 데이터베이스 클러스터로 승격시킨다.
    * 데이터베이스 스냅샷보다는 시간이 많이 걸리고 복제본 생성과 관련한 네트워크 비용도 발생할 수 있다.
* 외부 PostgreSQL 데이터베이스를 Aurora PostgreSQL로 옮겨야 하는 경우
  * 방법 1) 백업을 생성하여 Amazon S3에 두고 데이터를 가져오기 위해 aws\_s3 Aurora 확장자를 사용해서 새로운 데이터베이스를 생성할 수 있다.
* DMS를 통해 PostgreSQL에서 Amazon Aurora로 지속적 마이그레이션이 가능하다.

### 온프레미스 마이그레이션

* Amazon Linux 2 AMI를 ISO 타입의 VM으로 다운로드 할 수 있다.
  * Oracle VM과 Microsoft Hyper-V에 해당하는 VMWare, KVM Virtual Box에서 이미지를 통해 VM을 생성할 수 있다.
  * 직접 VM을 통해 온프레미스 인프라에서 Amazon Linux 2를 실행할 수 있다.
* VM 가져오기와 내보내기
  * 기존의 VM과 애플리케이션을 EC2로 마이그레이션 할 수 있다.
  * 온프레미스 VM들을 위한 DR Repository 전략을 생성할 수 있다.
  * 온프레미스 VM이 많은 경우 이를 클라우드에 백업할 수 있고, VM을 EC2에서 온프레미스 환경으로 다시 빼올 수도 있다.
* AWS Application Discovery Service
  * 온프레미스의 정보를 모아주고 마이그레이션을 계획할 수 있게 해 주는 서비스
  * 서버 사용량 정보와 종속성 매핑에 대한 정보를 제공하는 상위 수준의 서비스이다.
  * 온프레미스에서 클라우드로 대량의 마이그레이션 할 때 유용하다.
  * AWS Migration Hub를 사용해서 모든 마이그레이션을 추적할 수 있다.
* AWS Data Migration Service
  * `온프레미스 -> AWS`, `AWS -> AWS`, `AWS -> 온프레미스` 복제를 지원한다.
* AWS Server Migration Service
  * 온프레미스의 라이브 서버들을 AWS로 증분 복제할 때 사용된다.
  * AWS로 볼륨을 직접 복제할 수 있다.<br>

## AWS Backup

* AWS 서비스 간의 백업을 중점적으로 관리하고 자동화할 수 있게 도와주는 완전 관리형 서비스
* 다양한 서비스를 지원한다.
  * Amazon EC2, EBS, Amazon S3, RDS 및 모든 데이터베이스 엔진
  * Aurora, DynamoDB, DocumentDB Amazon Neptune, EFS, Lustre와 Windows 파일 서버를 포함하는 FSx
  * AWS Storage Gateway의 볼륨 게이트웨이
* 리전 간 백업을 지원하여 한 곳의 재해 복구 전략을 다른 리전에 푸시할 수 있다.
* 계정 간 백업을 지원하여 AWS에서 여러 계정을 사용할 경우에 유용하다.
* Aurora와 같은 지정 시간 복구(PITR)를 지원한다.
* 온디맨드 혹은 예약 백업을 지원한다.
* 태그 기반 백업 정책을 통해 특정 태그가 지정된 리소스만 백업할 수 있다.
* 백업 플랜(Plan)
  * 백업 빈도를 정의하여 특정 주기 또는 cron 표현식을 기반으로 백업 기간을 지정할 수 있다.
  * 일정 기간 후 백업을 콜드 스토리지로 이전할지 여부를 설정할 수 있다.
  * 백업 보유 기간을 지정할 수 있다.
* AWS Backup 과정
  * 백업 플랜을 생성한다.
  * 특정 AWS 리소스를 할당한다.
  * 데이터가 자동으로 AWS Backup에 지정된 내부 S3 버킷에 백업된다.
* 볼트(Vault) 잠금
  * WORM(Write Once Read Many) 정책을 시행하면 백업 볼트(Vault)에 저장한 백업을 삭제할 수 없다.
  * 의도치 않거나 악의적인 삭제 작업을 막고 백업 유지 기간 축소 또는 변경 작업을 방지할 수 있다.
  * 루트 사용자 자신도 백업을 삭제할 수 없게 된다.

## Application Discovery Service

* 온프레미스 서버나 데이터 센터의 정보를 수집하여 클라우드로 마이그레이션하는 계획을 만든다.
* 서버를 스캔하고 마이그레이션에 중요한 서버 설치 데이터 및 종속성 매핑에 대한 정보를 수집한다.
* 마이그레이션 방식은 크게 두 가지로 나뉜다.
  * Agentless Discovery
    * AWS Agentless Discovery Connector를 통해 VM, 설정, CPU와 메모리 및 디스크 사용량과 같은 성능 기록에 대한 정보를 제공한다.
  * Agent-based Discovery
    * Application Discovery 에이전트
    * 가상 머신 내에서 더 많은 업데이트와 정보를 얻을 수 있다.
    * 예를 들어, 시스템 구성, 성능, 실행 중인 프로세스, 시스템 사이의 네트워크 연결에 대한 세부 정보 등을 얻을 수 있다.
* 결과 데이터는 AWS Migration Hub라는 서비스에서 볼 수 있다. 이동해야 할 항목들이 내부적으로 어떻게 상호 연결되어 있는지 파악하기에 유용하다.
* AWS Application Migration Service
  * 온프레미스에서 AWS로 이동하는 가장 간단한 방법
  * Lift-and-shift 솔루션을 통해 물리적, 가상, 또는 다른 클라우드에 있는 서버를 AWS 클라우드 네이티브로 실행시킬 수 있다. (rehost)
  * 예를 들어 디스크에서 실행되는 프로그램이 있고 Application Migration Service를 실행한 경우, 데이터 센터에 설치된 복제 에이전트가 디스크를 저비용 EC2 인스턴스, EBS 볼륨으로 연속적으로 복제한다.
  *   컷오버를 수행할 준비가 되면, 저비용 스테이징에서 프로덕션으로 데이터를 이동(컷오버)시킨다. 즉, 데이터를 복제한 다음 특정 시점에서 컷오버를 수행한다.

      <figure><img src="../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>
* 광범위한 플랫폼, 운영 체제, 데이터베이스를 지원하고, 최소 다운타임을 보장한다. 별도 엔지니어가 필요 없으므로 비용이 절감된다.

## 대규모 데이터 셋을 AWS로 전송

* 데이터셋 크기가 매우 크고 네트워크 대역폭이 낮은 경우 다음 두 방식을 이용하면 한참 걸려 전달될 것이다.
  * 공용 인터넷을 사용
  * 사이트 간 VPN을 설치하여 빠르게 바로 연결
* Direct Connect를 통해 1Gbps로 프로비저닝하는 경우, 초기 설치에 시간이 오래 걸린다.
* Snowball을 여러 개 주문하여 약 일주일 대기 후, Snowball에 데이터를 로드하고 AWS로 보내면 종단 간 전송에 일주일 넘게 걸린다.
* Snowball을 통해 전송되고 있던 데이터베이스는 DMS와 결합하여 나머지 데이터를 전송할 수 있다.
* 지속적 복제에 대해서는 Site-to-Site VPN, Direct Connect나 DMS, DataSync 등의 서비스를 사용할 수 있다.

## VMware Cloud on AWS

* 온프레미스에 데이터 센터가 있을 때 vSphere 기반 환경과 가상 머신을 VMWare Cloud를 통해 관리할 수 있다.
* 이 때 데이터 센터의 용량을 확장하고 클라우드와 AWS를 모두 사용하고 싶은 경우, 전체 VMware Cloud의 인프라를 AWS에서 확장함으로써 vSphere, vSAN, NSX 등에서 사용할 수 있다.
* 예를 들어 컴퓨팅 성능을 확장하여 데이터 센터에서 클라우드뿐 아니라 스토리지까지 컴퓨팅이 가능해져 VMWare 기반 워크로드를 AWS로 마이그레이션할 수 있다.
* 프로덕션 워크로드를 여러 데이터 센터 간 실행할 수 있고 프라이빗, 퍼블릭, 하이브리드 클라우드 환경 모두 구축할 수 있다.
* 재해 복구 전략으로 활용 가능하다.
* Amazon EC2, Amazon FSx, S3, RDS Direct Connect, Redshift 등의 서비스를 사용할 수 있다.
