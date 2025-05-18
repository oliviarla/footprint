# Data Migration

## Disaster Recovery

* 온프레미스 간 재해 복구
  * 캘리포니아와 시애틀처럼 각 지역에 데이터 센터를 두고 사용하는 전형적인 재해 복구 유형으로 비용이 아주 많이 든다.
* 하이브리드 복구
  * 온프레미스를 기본 데이터 센터로 두고 재해 발생 시 클라우드를 사용하는 방식이다.
* 완전 클라우드 복구
  * AWS Cloud 내부의 여러 리전을 두어 재해 복구를 수행할 수 있다.
*   RPO vs RTO

    <figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

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



## RDS and Aurora Migration



## AWS Backup



## MGN



## 대규모 데이터 셋을 AWS로 전송



## VMware Cloud on AWS





