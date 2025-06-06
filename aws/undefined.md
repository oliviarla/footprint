# 기타 서비스

## CloudFormation

* 선언적 방법으로 AWS 리소스를 구성할 수 있는 기능을 제공하는 서비스
* 리소스 생성 및 삭제가 쉬워지며, 생산성이 증대된다.
  * 예를 들어 보안 그룹이 필요하고, 보안 그룹을 사용할 EC2 인스턴스 두 개가 필요하고, S3 버킷도 필요하고, 로드 밸런서가 필요한 경우, 사용자가 코드를 작성하면 CloudFormation은 리소스를 자동으로 생성한다.
* 직접 선후 관계를 파악하여 어떻게 동작해야 하는지 지정하는 명령형 방법이 아닌, 선언적 방법으로만 지정하면 AWS가 알아서 관계를 파악하고 리소스를 생성해준다.
* 코드를 변경할 때마다 코드 리뷰를 통해 검토할 수 있다.
* 스택 내의 각 리소스는 스택 내에서 만들어진 다른 리소스들과 비슷하게 태그되어 비용 효율적이고, 리소스 비용을 쉽게 예측할 수 있다.
* 자동으로 수행되는 전략을 통해 절약 전략을 세울 수 있다. 예를 들어 어떤 환경에서 매일 오전 9시에 템플릿을 생성하고 오후 5시에는 모든 템플릿을 삭제하도록 할 수 있다.
* AWS 상에서 지원되지 않는 리소스에 대해서는 사용자 지정 리소스를 사용할 수 있다.
* CloudFormation 템플릿을 시각화하기 위해 애플리케이션 컴포저 서비스를 사용할 수 있다. 이를 통해 CloudFormation 템플릿의 모든 리소스를 볼 수 있다. 모든 구성 요소 간의 관계 역시 확인 가능하다.
* 다른 환경, 지역 또는 다른 AWS 계정에서 아키텍처를 반복해야 할 때 사용하면 좋다.

### 서비스 역할

* CloudFormation이 스택 리소스를 직접 생성, 업데이트 및 삭제하기 위한 권한을 제공하는 IAM 역할
* CloudFormation 템플릿을 정의했을 때 사용자의 IAM 권한을 이용해 Cloudformation에서 작업을 수행할 수 있다.
* 서비스 역할을 생성해야 하는 유저는 반드시 iam:PassRole 권한을 가져야 한다.
* 서비스 역할에는 다양한 권한을 부여할 수 있는데, 예를 들면 버킷을 만들고, 업데이트하고, 삭제하는 등의 s3.\*Bucket 권한을 부여할 수 있다.
* 서비스 역할 사용 시 최소 권한 원칙을 지키고, 사용자에게 스택 리소스를 만들 수 있는 모든 권한을 부여하지 않고, Cloudformation에서 서비스 역할을 호출할 수 있는 권한만 부여하게 된다.

## Amazon SES

* Simple Email Service
* 완전 관리형 서비스로, 이메일을 전 세계에 대규모로 안전하게 보낼 수 있는 서비스이다.
* 애플리케이션을 SES API 또는 SMTP 서버와 연동하면 Amazon SES가 사용자들에게 대량으로 이메일을 보낸다.
* 아웃바운드뿐 아니라 인바운드 이메일도 허용하기 때문에 답장을 주고받을 수 있다.
* 이메일을 열었는지, 스팸 처리 했는지 여부를 알려주는 평판 대시보드, performance insights, 스팸 방지 피드백 기능을 제공한다.
* 이메일 전달 여부, 피드백 루프 결과 등 이메일 통계를 제공한다.
* 이메일 전송에 대한 최신 보안 기준(DKIM과 SPF 기능)을 지원한다.
* 유연한 IP 배포가 가능하다. 공유 IP, 전용 IP, 또는 고객 소유 IP들을 사용할 수 있다.
* AWS 콘솔, AWS API 혹은 SMTP 프로토콜을 사용해 서비스를 사용할 수 있다.
* 이메일 트랜잭션, 마케팅 이메일 대량 이메일 커뮤니케이션에 사용될 수 있다.

## Amazon Pinpoint

* Amazon Pinpoint는 확장 가능한 양방향 인바운드 및 아웃바운드 마케팅 커뮤니케이션 서비스이다.
* 이메일, SMS, 푸시 알림, 음성, 인앱 메시지를 보낼 수 있다.
* 고객에게 적합한 콘텐츠로 메시지를 세분화하고 개인화할 수 있다.
* 그룹이나 세그먼트 등의 단위를 만들 수 있다.
* 마케팅 이메일을 대량으로 보내거나 트랜잭션 SMS 메시지를 전송하여 캠페인을 실행할 때 사용될 수 있다.
* 전달 성공하거나 응답이 오면 TEXT\_SUCCESS, TEXT\_DELIVERED, REPLIED와 같은 이벤트가 Amazon SNS, Kinesis Data Firehose, CloudWatch Logs로 전달된다.
* Amazon SNS나 Amazon SES와의 차이점은 직접 각 메시지의 대상, 내용, 전달 일정을 관리하지 않아도 된다는 것이다.

## SSM Session Manager

* EC2 인스턴스와 온프레미스 서버에서 사용 가능한 보안 셸을 제공한다.
* 별도의 SSH 액세스, 배스천 호스트, SSH 키가 필요없다. 따라서, EC2 인스턴스의 포트 22가 닫혀있어도 문제 없다.
* EC2 인스턴스에 SSM 에이전트를 두면 에이전트가 세션 관리자 서비스와 연결된다. 사용자는 세션 관리자 서비스를 통해 EC2 인스턴스에 접근 가능하다.
* Linux, MacOS, Windows를 지원하며, 세션 기록의 로그 데이터를 Amazon S3 또는 CloudWatch Logs로 보낼 수 있다.
* AmazonSSMManagedInstanceCore 권한을 갖는 IAM 인스턴스 프로필을 생성해 인스턴스에 결합시키면, EC2가 이 정책을 사용하여 SSM 서비스와 연동된다.
* Fleet Manager에는 SSM에 등록된 모든 EC2 인스턴스가 표시된다.
* 결국 EC2 인스턴스에 접근하는 방법은 1. 포트 22를 연 다음 SSH 키와 터미널을 사용하여 SSH 명령을 수행하거나 2. 포트 22를 연 다음 EC2 Instance Connect를 사용하거나, 3. EC2 인스턴스가 세션 관리자를 사용할 수 있도록 허용한 후 보안 셸을 실행하는 것이다.

## SSM 기타 서비스

### 명령 실행

* 스크립트를 실행하거나 하나의 명령을 실행할 수 있다.
* 리소스 그룹을 사용하여 명령을 여러 인스턴스에서 실행할 수 있다.
  * 에이전트가 시스템 관리자 서비스에서 실행되고 있으므로, 시스템 관리자 서비스에 등록된 EC2 인스턴스 또는 온프레미스 서버들에서 명령을 수행할 수 있다.
* 세션 관리자와 유사하게 SSH가 필요없다. 명령은 SSM Agent의 명령 기능을 통해 EC2 인스턴스 또는 온프레미스 서버에서 수행된다.
* 실행된 명령의 결과는 모두 S3 또는 CloudWatch 로그로 보내진다.
* IAM, CloudTrail과 통합되어 보안을 제공하고 누가 어떤 명령을 실행하는지 확인할 수 있다.
* EventBridge로 실행 명령을 직접 실행할 수 있다.&#x20;
* 상태 알림을 Amazon SNS로 전송할 수 있다.

### 패치 관리자

* 인스턴스 관리 과정 패칭을 자동화하는 데 사용된다.
* EC2 인스턴스와 온프레미스 서버에서 운영 체제 및 애플리케이션 업데이트, 보안 업데이트를 적용할 수 있다.
* Linux, Mac, Windows를 지원한다.
* 즉시 패치 작업을 진행하거나 유지 관리 기간을 통해 패치 일정을 예약할 수 있다.
* 인스턴스를 스캔하여 패치 규정 준수 보고서를 생성할 수 있다. 이를 통해 모든 인스턴스가 올바르게 패치되었는지 확인하고패치가 누락된 사람이나 경우가 있는지 확인 가능하다.
* AWS Console, SDK, 유지 관리 기간에서 AWS-RunBatchBaseline 실행 명령을 통해 패치 관리자를 호출하면 패치 작업이 진행된다.
* 유지 관리 기간
  * 인스턴스에서 작업을 수행할 일정을 정의하는 데 사용된다.
  * 언제 얼마 동안 수행할지, 어느 인스턴스에 유지 관리 기간을 적용할지, 어떤 작업을 실행할 지 지정해야 한다.

### 자동화

* EC2 인스턴스 또는 AWS 리소스에서의 명령 유지 관리 및 배포 작업을 단순화하는 데 사용된다.
* ECC2 인스턴스들을 한 번에 다시 시작하거나 AMI를 생성하거나 EBS 스냅샷을 생성하는 등의 작업을 할 수 있다. 혹은 RDS 데이터베이스의 스냅샷을 생성할 수 있다.
* Automation Runbook이란 EC2 인스턴스 또는 AWS 리소스에 수행할 작업을 정의해둔 SSM Document이다.
* 다음 서비스들을 사용해 트리거될 수 있다.
  * 콘솔, SDK, CLI
  * Amazon EventBridge
  * 유지 관리 기간에 의한 실행 주기
  * AWS config
    * AWS config에서 리소스가 준수되지 않은 것을 알게 된 경우, SSM 자동화를 실행하기 위해 자동으로 수정 조치를 취할 수 있다.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## AWS 비용 탐색기

* AWS 비용 및 시간에 따른 사용량을 시각화하고 이해하며 관리하는 데 사용된다.
* 사용자 정의 보고서를 생성해서 비용과 사용량 데이터를 분석할 수 있다.
* 전체 계정 간 총비용 및 사용량 등 전체적인 데이터 분석도 가능하고, 매달, 매시간 혹은 리소스 레벨의 세부 단위로 분석 가능하다.
* 청구서 요금을 낮추기 위한 최적의 절감형 플랜(Saving Plan)을 선택하도록 도와준다.
  * 절감형 플랜은 예약 인스턴스의 대안이 될 수 있으며 비용 탐색기가 알아서 추천 플랜을 보여주기도 한다.
* 과거에 지출했던 비용을 기반으로 향후 12개월까지의 사용량을 예측할 수 있다.
* 월별 비용, 시간별 리소스 레벨을 확인할 수 있다.

## AWS 비용 이상 탐지

* 비용 및 사용량 데이터를 지속적으로 모니터링한다.
* 과거 패턴을 학습한 기계 학습 모델을 통해 비정상적인 범위를 감지한다.
* 일회성 비용 급증 또는 지속적인 비용 증가를 감지한다.
* AWS 서비스, 회원 계정, 비용 할당 태그 및 비용 범주를 모니터링하고 계정에서 발생하는 문제에 대한 근본 원인 분석이 포함된 이상 탐지 보고서를 만든다.
* 개별 알림이나 SNS를 활용한 일간 또는 주간 요약을 통해 알림을 받을 수 있다.

## AWS Outposts

* Hybrid Cloud
  * 온프레미스 인프라를 유지하는 기업이 클라우드 인프라도 함께 운영하는 방식을 의미한다.
  * AWS 클라우드는 AWS Console, CLI, AWS API를 통해 사용하고, 온프레미스는 자체 시스템을 통해 사용해야 한다.
* Outposts는 AWS가 기업의 온프레미스 데이터센터에 서버 랙 형태로 제공하는 완전 관리형 서비스이다. AWS 인프라 서비스, API, Tools를 제공하여 on-premise에서 애플리케이션을 클라우드처럼 구축할 수 있다.
* AWS가 직접 Outposts 렉을 설치하고 관리하면서 사용자의 온프레미스 인프라에서 AWS 클라우드를 운영할수 있도록 해준다.
* 렉에 존재하는 서버들은 AWS 서비스가 사전 설치된 상태로 제공된다.
* 보안 책임이 기업에 전가된다.
* 저지연 엑세스로 on-premise 시스템에 접근 가능하고 로컬 데이터 처리가 가능하므로 데이터가 온프레미스 환경을 벗어나지 않는다.
* 데이터는 데이터 센터 내에 저장된다.
* on-premise에서 Outposts로 마이그레이션하기에 쉽다.
* Amazon EC2, EBS, S3, EKS, ECS, RDS, EMR등 다양한 서비스를 사용할 수 있다.

## AWS Batch

* 완전 관리형 배치 처리 서비스
* 어떤 규모의 배치라도 처리할 수 있다.
* AWS에서 수십만 개의 컴퓨팅 배치 작업을 매우 쉽고 효율적으로 실행할 수 있다.
* 배치 작업에는 시작과 끝 시점이 있다. 따라서 Batch 서비스는 동적으로 EC2 인스턴스 또는 스팟 인스턴스를 시작한다.
* Batch는 배치 대기열을 처리할 수 있도록, 적절한 양의 컴퓨팅 및 메모리를 프로비저닝한다. EC2 인스턴스 또는 스팟 인스턴스의 적절한 수를 자동으로 조정해주기 때문에 비용이 최적화되고 인프라에 신경쓸 필요가 없다.
* 배치 작업을 배치 대기열에 올리거나 예약하기만 하면, Batch 서비스가 나머지 작업을 수행한다.
* 도커 이미지와 ECS 서비스에서 실행할 수 있는 배치 작업을 정의해야 한다.
* Lambda와의 차이점
  * Lambda는 서버리스이며 15분의 시간 제한이 있으며 일부 프로그래밍 언어 몇 개로만 사용 가능하다. 또한 작업을 실행하는 임시 디스크 공간이 제한되어 있다.
  * Batch는 관리형 서비스이지만 EC2 인스턴스에 의존하기 때문에 시간 제한이 없다. 도커 이미지로 패키징하면, 런타임의 길이는 상관없다. EBS, EC2 인스턴스 스토어 등의 디스크에 의존할 수 있다.&#x20;

## Amazon AppFlow

* SaaS 애플리케이션 및 AWS 사이에 데이터를 전송할 수 있는 완전 관리형 통합 서비스
* 데이터 소스로는 Salesforce, SAP, Zendesk, Slack, ServiceNow 등이 있다.
* Amazon S3, Amazon Redshift 같은 AWS 서비스나 Snowflake, Salesforce 등에 데이터를 내보낼 수 있다.
* 주기를 지정하거나 특정 이벤트에 대한 응답, 온디맨드에 따라 통합할 수 있다.
* 필터링, 유효성 검사와 같은 데이터 변환이 가능하다.
* 공용 인터넷을 통해 암호화되거나, PrivateLink를 사용하여 비공개로 전송할 수 있다.
* API를 활용하여 계정 내에서 바로 데이터를 사용할 수 있으므로 별도의 통합이 필요 없다.

## AWS 증폭

* 웹 및 모바일 애플리케이션 개발 도구
* AWS의 많은 스택을 한 곳에 통합해서 웹 및 모바일 애플리케이션을 빌드할 수 있다.
* Amplify CLI를 사용해서 Amplify 백엔드를 만들고, Amplify 프론트엔드 라이브러리를 추가해 통합할 수 있다.
* Amplify 백엔드에서는 내부적으로 다양한 AWS 서비스를 사용할 수 있다. 한 곳에서 인증, 스토리지, REST API나 GraphQL API, CI/CD, PubSub, Analytics, AI/ML 예측, 모니터링 등을 설정할 수 있다.
* Github, AWS CodeCommit 등으로부터 소스 코드를 연동할 수 있다.
* 다양한 웹/모바일 애플리케이션용 프런트엔드 라이브러리, 프레임워크가 제공된다.
* Amplify 콘솔을 이용해서 Amplify 자체에 배포하거나 Amazon CloudFront에 배포하여 웹 또는 모바일 애플리케이션을 제공할 수 있다.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## 인스턴스 스케줄러

* CloudFormation을 통해 배포되는 AWS 솔루션이다.
* 이 솔루션을 CloudFormation을 사용하여 배포하면 계정 내에서 AWS 서비스를 자동으로 시작하고 중지하여 비용을 최대 70%까지 절감할 수 있는 기능을 제공한다.
* 예를 들어 회사의 EC2 인스턴스를 업무 시간 외에 중지하고 싶다면 AWS의 Instance Scheduler를 통해 EC2 Auto Scaling Groups 및 RDS 인스턴스를 지원한다.
* 모든 스케줄은 DynamoDB 테이블에서 관리되며, Lambda 함수가 DynamoDB에서 일정을 조회한 후 다른 람다를 트리거하여 필요한 서비스의 인스턴스를 자동으로 중지하거나 시작하는 과정이 내부적으로 수행된다.
* 계정 간 및 지역 간 리소스를 지원한다.
