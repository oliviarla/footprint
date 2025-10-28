# Monitoring

## CloudWatch

* AWS의 모든 서비스에 대한 지표를 제공한다.
* EC2 인스턴스의 지표로는 CPUUtilization, NetworkIn 등이 있고, Amazon S3의 지표로는 버킷 크기 등이 있다.
* 지표(metric)의 특성
  * 지표는 namespaces에 속하며, 각기 다른 namespaces에 저장된다. 서비스당 namespaces은 하나이다.
  * 지표의 속성으로 측정 기준(Dimension)이 존재하며 최대 30개까지 가질 수 있다.
    * 예를 들어 CPU 사용률에 대한 지표는 특정 인스턴스 ID나 특정 환경 등과 관련이 있을 수 있다.
  * 지표는 시간을 기반으로 하므로 타임스탬프가 존재한다.
  * 지표가 많아지면 CloudWatch 대시보드에 추가해 모든 지표를 한 번에 볼 수 있다.
  * CloudWatch 사용자 지정 지표를 만들어 사용할 수 있다. 예를 들어 EC2 인스턴스로부터 메모리 사용량을 추출하여 지표로 만들 수 있다.
  * CloudWatch 외부로 스트리밍할 수 있다. 원하는 대상으로 지속적으로 스트리밍하면 거의 실시간으로 전송되고 지연 시간도 짧아진다.
    * Amazon Kinesis Data Firehose로 거의 실시간으로 스트리밍 가능하다. Firehose에서는 S3 버킷으로 지표를 보낸 후 Athena에서 분석하도록 하거나, Redshift나 OpenSearch에 보낼 수 있다.
    * Datadog, Dynatrace New Relic, Splunk, Sumo Logic 같은 서드파티 서비스 제공자에 CloudWatch 지표를 직접 전송할 수도 있다.
  * 특정 네임스페이스의 지표만 필터링할 수 있다.
* 인스턴스에 세부 모니터링을 활성화하면 1분 단위의 데이터를 얻을 수 있다.
* 원하는 리전을 기반으로 혹은 측정 기준을 기반으로 지표를 한눈에 볼 수도 있고 리소스 ID로 필터링할 수도 있다.

### CloudWatch Logs

* 로그 그룹을 직접 정의한다. 로그 그룹 안에는 다수의 로그 스트림이 존재한다. 애플리케이션 안의 로그 인스턴스나 특정 로그 파일 또는 컨테이너를 나타낸다.
* 로그 만료 정책을 정의하여 언제 만료될 지 정의해야 한다. 영원히 만료되지 않고 보관할 수도 있다.
* 배치 형태로 Amazon S3에 내보내거나, Kinesis Data Streams, Kinesis Data Firehose, AWS, 람다, Amazon OpenSearch에 스트리밍할 수 있다.
* 모든 로그는 기본적으로 암호화된다. 필요 시 자체적으로 KMS 기반 암호화를 설정할 수도 있다.
*   다음과 같은 데이터 소스를 활용할 수 있다.

    * SDK, CloudWatch Logs Agent나 CloudWatch Unified Agent를 소스로 사용할 수 있다.

    > CloudWatch Unified Agent는 CloudWatch Logs Agent를 대체하였다.

    * Elastic Beanstalk을 사용한 애플리케이션에서 수집된 로그를 사용할 수 있다.
    * ECS는 로그를 컨테이너에서 곧바로 CloudWatch로 보낼 수 있다.
    * 람다는 함수의 로그를 전송한다.
    * VPC Flow Logs는 VPC 네트워크 트래픽의 로그를 전송한다.
    * API Gateway는 모든 요청을 CloudWatch Logs로 보낼 수 있다.
    * 필터링 기반의 CloudTrail 로그를 전송할 수 있다.
    * Route 53은 서비스에 대한 모든 DNS 쿼리를 로깅한다.
* CloudWatch Logs Insights
  * CloudWatch Logs에 보관된 로그를 쿼리할 수 있는 대시보드이다.
  * 이를 통해 로그 데이터를 검색하고 분석할 수 있다.
  * 쿼리를 적용하려는 시간대를 지정하고, 쿼리를 보낸 후 시각화하여 결과로 내보내거나 대시보드에 추가해 원할 때마다 다시 실행할 수 있다.
  * 다양한 샘플 쿼리를 제공한다. 예를 들면 25개의 최신 이벤트를 검색하거나 얼마나 많은 이벤트에 예외나 오류가 있었는지, 특정한 IP가 포함된 로그를 검색할 수도 있다.
  * 조건을 기준으로 필터링하거나 집계 계산, 이벤트 정렬, 이벤트 개수 제한 등의 작업을 할 수 있다.
  * 다른 계정의 로그 그룹을 포함한 다수의 로그 그룹을 쿼리할 수 있다.
  * 실시간 엔진이 아니므로 쿼리를 실행하면 과거 데이터만 쿼리하게 된다.
* 다음과 같이 로그를 내보낼 수 있다.
  * Amazon S3
    * 모든 로그를 Amazon S3로 전송하는 배치 내보내기 기능을 제공한다. 배치 방식이므로 실시간이나 근 실시간이 아니다.
    * 완료될 때까지 최장 12시간까지 걸릴 수 있다.
    * 내보내기를 시작하기 위한 API 호출을 CreateExportTask라고 한다.
  * CloudWatch Logs Subscription
    * 로그 이벤트들의 실시간 스트림을 제공한다.
    * 데이터를 Kinesis Data Streams, Kinesis Data Firehose 또는 람다 등의 다양한 곳에 전송하여 처리 및 분석이 가능하다.
    * Subscription Filter를 써서 전송 대상으로 전달하려는 로그 이벤트의 종류를 지정할 수 있다.
    * 로그 데이터를 Kinesis Data Streams, Kinesis Data Firehose, 람다에 보낼 수 있다.
    * Kinesis Data Streams에 전달된 로그는 Kinesis Data Analytics 또는 Amazon EC2 또는 람다와 통합될 수 있다.
    * Kinesis Data Firehose에 저장된 데이터는 S3에 거의 실시간으로 저장되거나 OpenSearch Service에 전달될 수 있다.
    * 각 계정에 Subscription Filter를 두고 하나의 Kinesis Data Streams에 데이터를 실시간으로 내보낼 수 있다.
    * 서로 다른 계정 간 로그를 전달할 수도 있는데 이를 Cross-Acount Subscription이라고 부른다.&#x20;
      * 발신자 계정에는 CloudWatch Logs Subscription Filter를 두고, 수신자 계정의 Kinesis Data Stream을 가상으로 표시하는 Subscription Destination으로 전송한다.&#x20;
      * Destination Access Policy를 통해 계정 간 로그 이동이 가능하도록 해야 한다.
      * Cross-Account IAM 역할을 통해 Kinesis Data Stream에 전송할 권한을 가진 IAM 역할을 생성해야 한다.

### Live Tail

* 로그 이벤트를 테일링하는 기능을 지원하는 서비스이다.
* 로그들이 매우 빠르게 스트림되고 있으면 Live Tail 화면에 전부 나타나므로, 언제 일어난 것인지, 그룹이 어떤 것인지 등을 알 수 있고 해당 로그 스트림이 일어난 링크로 바로 넘어갈 수 있다.
* 이를 통해 CloudWatch 로그를 쉽게 디버깅 할 수 있다.
* 일시적으로 몇 시간만 사용하여 디버깅을 하는 용도이다. 필요한 상황이 아니라면 세션을 닫아 비용 부과를 줄일 수 있다.

### Agent

* **EC2 인스턴스에서 발생한 로그를 CloudWatch로 옮기려면 EC2 인스턴스에 에이전트 프로그램을 실행시켜 원하는 로그 파일을 푸시**해야 한다.
* EC2 인스턴스에는 로그를 보낼 수 있게 해주는 IAM 역할이 있어야 한다.
* 에이전트는 온프레미스 환경에서도 사용 가능하다.
* CloudWatch Logs Agent
  * CloudWatch Logs로 로그를 보내는 기능만 제공한다.
* CloudWatch Unified Agent
  * 프로세스나 RAM 같은 추가적인 시스템 단계 지표를 수집하여 CloudWatch Logs에 로그를 보낸다.
  * SSM Parameter Store를 이용해서 에이전트를 쉽게 구성할 수 있다. 모든 통합 에이전트를 대상으로 중앙 집중식 환경을 구성할 수 있다.
* 지표
  * 리눅스 서버나 EC2 인스턴스의 지표를 직접 수집할 수 있다.
  * CPU, Disk, RAM, Netstat, Processes, Swap Space 등을 수집할 수 있다.
  * 기본 EC2 인스턴스 모니터링 보다 세부 지표를 얻고 싶다면 통합 CloudWatch 에이전트를 이용하면 된다.

### Alarms

* 어떠한 메트릭에 대해서든 알림을 생성할 수 있다.
* 다양한 기준을 사용할 수 있다.
* 다음과 같은 상태를 가진다.
  * OK: 트리거되지 않은 상태
  * INSUFFICIENT\_DATA: 데이터가 부족해 파악하기 어려운 상태
  * ALARM: 임계값이 초과되어 알람이 활성화된 상태
* 특정 지표를 특정 기간을 기준으로 하여 판별할 수도 있다. 예를 들어 10초, 30초, ... 등을 지정 가능하다.
* 알람이 트리거되면 **알람 액션을 통해 EC2 인스턴스를 정지/종료/재부팅할 수도 있고, Auto Scaling을 수행할 수도 있고, Amazon SNS 서비스에 알림 정보를 보내 람다와 통합할 수도 있다.**
* 여러 지표를 복합적인 기준으로 삼아 알람을 보내려면 Composite Alarms를 사용해야 한다.
  * 각각 다른 지표를 기반으로 and 또는 or 조건을 사용할 수 있다.
  * 내부에 복잡한 composite alarms를 만들면 alarm noise를 크게 줄일 수 있다. 예를 들어, CPU 사용량이 많고 네트워크 사용량도 높다면 알람을 보내지 않도록 설정할 수 있다.
* EC2 Instance Recovery
  * CloudWatch alarm이 발동 시 EC2 인스턴스의 상태를 검사한다.
  * System Status Check를 통해 EC2 instance가 실행되는 하드웨어를 확인한다.
  * Attached EBS Status Check는 EC2 instance에 연결된 EBS 볼륨 상태를 확인한다.
  * Recovery를 진행하면 동일한 private, public, elastic IP와 메타데이터를 그대로 유지하며 동일한 placement group을 설정할 수 있으며 또한 SNS와 연동하여 알람을 보낼 수 있다.
* CloudWatch logs metric filter를 통해 특정 이벤트를 감지하고 알람을 생성할 수 있다.
* 알람 기능을 테스트하기 위해 `set-alarm-state`라는 CLI 명령어를 사용할 수 있다.

```
ws cloudwatch set-alarm-state --alarm-name "myalarm" --state-valueALARM --state-reason "testing purposes"
```

### Insight

* CloudWatch Container Insights
  * 컨테이너로부터 지표와 로그를 수집, 집계, 요약하는 서비스
  * Amazon ECS, Amazon EKS, EC2에서 실행되는 Kubernetes 플랫폼, ECS와 EKS의 Fargate에 배포된 컨테이너에서 사용 가능하다.
  * 컨테이너로부터 지표와 로그를 손쉽게 추출해서 CloudWatch에 세분화된 대시보드를 만들 수 있다.
  * **컨테이너화된 버전의 CloudWatch 에이전트를 사용해야 컨테이너를 찾을 수 있다.**
* Lambda Insights
  * AWS Lambda에서 실행되는 서버리스 애플리케이션을 위한 모니터링과 트러블 슈팅 솔루션
  * CPU 시간, 메모리 디스크, 네트워크 등 시스템 수준의 지표를 수집, 집계, 요약한다.
  * 람다 레이어에서 제공되어 Lambda 함수의 성능을 모니터링한다.
* Contributor Insights
  * 컨트리뷰터 데이터를 표시하는 시계열 데이터를 생성하고 로그를 분석하는 서비스
  * 상위 N개의 컨트리뷰터나 총 컨트리뷰트 수 및 사용량을 볼 수 있다.
  * 시스템 성능에 영향을 미치는 대상을 파악할 수 있다.
    * 예를 들어 사용량이 많은 네트워크 사용자를 식별하려면, 모든 네트워크 요청에 대해 VPC 내에서 생성되는 로그인 VPC 플로우 로그가 CloudWatch Logs로 전달되고, CloudWatch Contributor Insights가 이를 분석하여  VPC에 트래픽을 생성하는 상위 10개의 IP 주소를 찾고 좋은 사용자인지 나쁜 사용자인지 판단한다.
  * VPC 로그, DNS 로그 등 AWS가 생성한 모든 로그에서 작동한다.
  * 규칙을 직접 생성하거나 AWS가 생성한 간단한 규칙을 활용하여 메트릭 분석에 활용할 수 있다. 규칙은 백그라운드에서 CloudWatch Logs를 활용해 적용된다. 내장된 규칙이 있어 다른 AWS 서비스에서 가져온 지표를 분석할 수 있다.
  * CloudWatch Application Insights
    * 자동화된 대시보드를 제공하여 모니터링하는 애플리케이션의 잠재적인 문제를 보여주고, 진행 중인 이슈와 분리할 수 있도록 도와준다.
    * Java나 .NET Microsoft IIS 웹 서버나 특정 데이터베이스를 선택해 선택한 기술로만 애플리케이션을 실행할 수 있다.
    * EBS, RDC, ELB ASG, Lambda, SQS, DynamoDB, S3 버킷, ECS 클러스터, EKS 클러스터, SNS 토픽, API Gateway와 같은 AWS 리소스에 연결된다.
    * 자동화된 대시보드를 생성할 때 SageMaker를 기반으로 생성된다.
    * 애플리케이션 상태 가시성을 높여 트러블 슈팅이나 애플리케이션을 보수하는 시간이 줄어든다.
    * 발견된 문제와 알림은 Amazon EventBridge와 SSM OpsCenter로 전달된다.

## EventBridge

* 예전에 CloudWatch Events라고 불렸다.
* Cron job 형태로 스크립트를 예약할 수 있다.
* 특정 작업을 수행하는 서비스에 반응하는 이벤트 패턴 기능을 제공한다.
  * 예를 들어 콘솔의 IAM 루트 사용자 로그인 이벤트가 발생하면 SNS 주제로 메시지를 보내고 이메일 알림을 받을 수 있다.
* Lambda 함수를 트리거해서 SQS, SNS 메시지 등을 보낼 수도 있다.
* 다양한 소스에서 생성되는 이벤트를 받아들인다.
  * EC2 Instance의 시작, 중단, 종료 이벤트, CodeBuild에서 실패한 빌드, S3에서 객체 업로드로 발생한 이벤트 등
  * **CloudTrail을 결합하면 AWS 계정에서 생성된 API 호출을 모두 가로채므로 아주 유용하다.**
* Amazon EventBridge로 전송되는 이벤트에는 필터를 설정할 수 있다.
* Amazon EventBridge는 이벤트의 세부 사항을 나타내는 JSON 문서를 생성하여 어떤 인스턴스가 이 ID로 실행되는지 등을 나타내고 시간, IP 등의 정보를 담는다.
* 이벤트들은 다양한 대상으로 전송할 수 있다.
  * Lambda 함수를 트리거하는 일정을 예약하거나 AWS Batch에서 배치 작업 예약을 하거나 Amazon ECS에서 ECS 작업을 실행할 수 있다.
  * SQS, SNS나 Kinesis Data Streams로 메시지를 보낼 수 있다.
  * 단계 함수를 실행할 수도 있고 CodePipeline으로 CI/CD 파이프라인을 시작하거나 CodeBuild에서 빌드를 실행할 수 있다.
* 이벤트 버스
  * AWS의 서비스는 이벤트를 **기본 이벤트 버스**로 전송한다.
  * **파트너 이벤트 버스**를 통해 Zendesk, Datadog, Auth0 등 AWS SaaS 파트너에서 이벤트를 전송할 수 있다. 이를 통해 AWS 외부에서 일어나는 변화에 반응할 수 있다.
  * **사용자 지정 이벤트 버스**를 통해 애플리케이션의 자체 이벤트를 사용자 지정 이벤트 버스로 전송할 수 있다.
  * 리소스 기반 정책을 사용해 다른 계정의 이벤트 버스에 액세스할 수 있다.
  * 모든 이벤트 혹은 필터링된 이벤트를 아카이빙할 수 있고, 보존 기간을 무기한이나 일정 기간으로 설정할 수 있다. 아카이브된 이벤트는 리플레이 가능하다.
* 스키마 레지스트리
  * Amazon EventBridge는 버스의 이벤트를 분석하고 스키마를 추론한다.
  * 스키마 레지스트리의 스키마를 사용하면 애플리케이션의 코드를 생성할 수 있고 이벤트 버스의 데이터가 어떻게 정형화되는지 미리 알 수 있다.
* 리소스 기반 정책을 통해 특정 이벤트 버스의 권한을 관리하여 특정 이벤트 버스가 다른 리전이나 다른 계정의 이벤트를 허용하거나 거부하도록 설정할 수 있다.

## CloudTrail

* AWS 계정 안에서 일어난 모든 이벤트와 API 호출 이력을 콘솔, SDK, CLI 또는 기타 AWS 서비스를 통해 얻을 수 있다.
* 로그들을 CloudTrail에서 CloudWatch Logs나 Amazon S3로 내보낼 수 있다. 이벤트를 최대 90일까지만 보관 가능하므로 그 이상 보관해야 한다면 이와 같이 이동시켜야 한다. S3로 이동된 로그는 Amazon Athena를 사용해 이벤트를 검색하고 분석할 수 있다.
* 모든 리전에 걸쳐 누적된 이벤트 이력을 하나의 특정한 S3 버킷에 넣고 싶으면, 트레일을 생성해서 모든 리전이나 하나의 리전에 적용할 수 있다.
* 관리 이벤트
  * AWS 계정 안에서 리소스에 대해 수행된 작업을 나타낸다.
  * 예를들어 보안을 설정할 때마다 그들은 IAM AttachRolePolicy라는 API 호출을 사용하면 해당 내역이 로깅되고,서브넷을 만들면 해당 내역이 로깅된다.
  * 리소스를 변경하지 않는 읽기 이벤트
    * 조회 성격의 API를 사용했을 때 생성되는 이벤트
    * 예를 들어 IAM에서 모든 사용자를 나열하거나 EC2에서 모든 EC 인스턴스를 나열하는 경우 발생한다.
  * 리소스를 수정할 수 있는 쓰기 이벤트
    * 변경 성격의 API를 사용했을 때 생성되는 이벤트
    * 예를 들어 DynamoDB 테이블을 삭제하거나 삭제 시도를 하는 경우 발생한다.
    * AWS 인프라에 손상을 줄 수 있어 중요하다.
* 데이터 이벤트
  * 고용량 작업이므로 기본적으로 로깅되지 않는다.
  * Amazon S3의 GetObject나 DeleteObject, PutObject, AWS 람다 함수 Invoke API 기록 등이 여기에 해당된다.
  * 읽기, 쓰기 이벤트를 분리할 수 있다.
* CloudTrail Insights 이벤트
  * 모든 유형의 서비스에 걸쳐 아주 많은 관리 이벤트가 일어나고 아주 많은 API 호출이 빠르게 이루어진다면 비정상적인 활동을 찾아내기 어렵다.
  * CloudTrail Insights를 사용하려면 비용을 지불해야 한다.
  * CloudTrail Insights는 계정에서 일어나는 이벤트를 분석하고 비정상적인 활동에 대한 탐지를 한다. 예를 들어 부정확한 리소스 프로비저닝, 서비스 한도 도달, AWS IAM 액션의 폭증, 주기적 유지보수 활동 누락 등을 탐지한다.
  * 정상적인 관리 활동을 분석해서 기준선을 만들고 올바른 형태의 이벤트인지 분석한다.
  * 관리 이벤트 중 쓰기 이벤트는 CloudTrail Insights에 의해 계속 분석되고 비정상적인 로그가 탐지되면 인사이트 이벤트가 생성된다. 이는 Amazon S3로 전달되고, EventBridge 이벤트를 생성한다.

## AWS Config

* AWS 내 리소스에 대한 감사와 규정 준수 여부를 기록할 수 있게 해주는 서비스
* 이를 통해 문제가 있을 경우 인프라를 빠르게 롤백하고 문제점을 찾아낼 수 있다.
* 예를 들어 다음과 같은 경우를 감지할 수 있다.
  * 보안 그룹에 제한되지 않은 SSH 접근이 있는지?
  * 버킷에 공용 액세스가 있는지?
  * 시간이 지나며 변화한 ALB 구성이 있는지?
* 규칙이 규정을 준수하든 아니든 변화가 생길 때마다 SNS 알림을 받을 수 있다.
* 리전별 서비스이기 때문에 모든 리전별로 구성해야 한다.
* 데이터를 중앙화하기 위해 리전과 계정 간 데이터를 통합할 수 있다.
* 모든 리소스의 구성을 S3에 저장해 추후 Athena 등 서버리스 쿼리 엔진을 통해 분석할 수 있다.
* AWS 관리형 Config 규칙을 통해 기본적으로 사용할 수 있는 규칙을 제공한다. AWS Lambda를 이용해 직접 Config 규칙을 만들 수도 있다.
* 어떤 동작을 미리 예방하지는 못한다.
* Config는 결제해야 하며 각 리전당 기록된 구성 항목별로 3센트를 지불해야 하고, 리전당 Config 규칙 평가별로 0.1센트를 내야 한다.
* 리소스의 규정 준수 여부, 시간별 리소스 설정 변경 내역을 조회할 수 있다. CloudTrail과 연결해 리소스에 대한 API 호출을 볼 수 있다.
* Remediations
  * SSM 자동화 문서를 이용해서 규정을 준수하지 않는 리소스를 수정하는 작업을 트리거할 수 있다.
  * 수정 작업을 통해 리소스를 자동 수정했음에도 여전히 규정을 미준수한다면 5번까지 재시도될 수 있다.

<figure><img src="../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

* Notifications
  * 리소스가 규정을 미준수했을 때마다 EventBridge를 사용해 알림을 보낼 수 있다.
  * 모든 구성 변경과 모든 리소스의 규정 준수 여부 알림을 Config에서 SNS 토픽으로 보낼 수 있다.
  * 한 설정에 대해 발생하는 여러 이벤트(설정 변경 / 규칙 준수 위반 등) 중 특정 이벤트에만 적용되게 필터링 하고 싶다면 SNS 필터링을 사용해 SNS 토픽을 필터링할 수 있다.
  * 하나의 SNS 토픽에만 스트리밍할 수 있다.

## ELB 적용 사례

* CloudWatch
  * 들어오는 연결 수를 모니터링하고 오류 코드 수를 시간 흐름에 따라 비율로 시각화할 수 있다.
  * 로드 밸런서의 성능을 볼 수 있는 대시보드를 생성 가능하다.
* Config
  * 로드 밸런서에 대한 보안 그룹 규칙을 추적해 수상하게 행동하거나 무언가 변경하는 행위를 추적한다.
  * SSL 인증서가 로드 밸런서에 항상 할당되어 있어야 한다는 규칙을 만들어 암호화되지 않은 트래픽이 로드 밸런서에 접근하지 못하게 할 수 있다.
* CloudTrail
  * 누가 API를 호출하여 로드 밸런서를 변경했는지 추적한다.
  * 누군가 보안 그룹 규칙을 바꾸거나 혹은 SSL 인증서를 바꾸거나 삭제한다면 CloudTrail이 자세한 정보를 알려준다.
