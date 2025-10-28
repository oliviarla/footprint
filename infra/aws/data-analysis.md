# Data Analysis

## Amazon Athena

* **Amazon S3 버킷에 저장된 데이터 분석**에 사용하는 서버리스 쿼리 서비스
* 데이터를 분석하려면 표준 SQL 언어로 파일을 쿼리해야 한다. 이를 위해 Athena는 SQL 언어를 사용하는 Presto 엔진에 빌드된다.
* S3 버킷에 업로드된 데이터를 다른 위치로 이동시키지 않고 바로 데이터를 쿼리하고 분석할 수 있다.
* CSV, JSON, ORC, Avro Parquet 등 다양한 형식을 지원한다.
* 스캔된 데이터의 TB당 고정 가격이 과금된다.
* 서버리스이므로 데이터베이스를 프로비저닝 할 필요가 없다.
* Amazon QuickSight라는 도구와 함께 사용하여 분석 결과를 보고서와 대시보드로 내보낼 수 있다.
* 임시 쿼리 수행이나 비즈니스 인텔리전스 분석 및 보고, VPC 흐름 로그, 로드 밸런서 로그, CloudTrail 추적 등을 분석할 때 사용된다.
* 성능 향상 방법
  * 데이터를 적게 스캔할 수 있는 데이터 타입을 사용한다. 열(column) 기반 데이터 유형을 사용하면 필요한 열만 스캔하므로 비용을 절감할 수 있다. 이를 위해 Apache Parquet과 ORC를 사용하면 된다.
  * 작은 크기의 데이터를 조회하기 위해 데이터를 압축해 저장해둘 수 있다.
  * 다음으로 특정 열을 항상 쿼리한다면 데이터셋을 파티셔닝할 수 있다. S3 버킷의 경로를 슬래시로 분할하여 열별로 특정 값을 저장할 수 있다. 이렇게 되면 데이터를 쿼리할 때 Amazon S3의 어떤 경로에 접근해 데이터를 조회할 지 알기 쉬워지고 적은 용량의 데이터만 조회 가능해진다.
  * 128MB가 넘는 큰 파일을 사용해서 오버헤드를 최소화할 수 있다. 파일이 클수록 스캔과 검색이 쉽기 때문이다.
* Federated Query
  * 람다와 연동하여 S3 외에 ElastiCache, DocumentDB, DynamoDB 등 다른 데이터 소스에 연결할 수 있다.
  * 각 데이터 소스로부터 얻은 쿼리 결과를 쿼리를 조인하거나 더 나은 데이터를 판별할 수 있다.
  * 쿼리 결과는 사후 분석을 위해 Amazon S3 버킷에 저장할 수 있다.

## Redshift

* 데이터베이스인 동시에 분석 엔진 기능을 제공한다.
* Redshift는 PostgreSQL을 기반으로 하여 SQL문을 사용해 쿼리를 수행할 수 있다. 단, PostgreSQL과 달리 OLTP 용도로 사용하지 않는다.

> OLTP: 온라인 트랜잭션 처리, 롤백을 지원하며 소규모의 데이터를 처리할 때 사용한다.\
> OLAP: 온라인 분석 처리, 데이터 웨어하우스 등의 시스템과 연관되어 데이터를 분석하고 복잡한 프로세싱을 수행할 수 있다.

* 다른 데이터 웨어하우스보다 10배 좋은 성능을 제공한다.
* PB단위로 확장 가능하다.
* 데이터를 로드하면 Redshift 내에서 바로 무작위화할 수 있다.
* 데이터를 Columnar(열 기반) 스토리지로 사용하여 성능이 좋다. 행 기반 스토리지와 달리 병렬 쿼리 엔진을 사용한다.
* Amazon QuickSights나 Tableau 같은 비즈니스 인텔리전스 툴과 통합 가능하다.
* Athena와 달리 **Amazon S3로부터 Redshift로 모든 데이터를 로드한 후 쿼리를 진행**한다. 따라서 조인과 집계 등 복잡한 쿼리를 빠르게 수행할 수 있다.
* Redshift에는 인덱스가 있고 고성능 데이터 웨어하우스를 위해 인덱스를 빌드한다.

#### 클러스터

* Redshift 클러스터에 공급한 인스턴스만큼 비용을 지불해야 한다.
* 쿼리를 계획하고 결과를 집계하는 리더 노드와 쿼리를 수행하고 결과를 리더에게 전송하는 계산 노드로 분리된다. 클러스터의 노드 크기는 미리 지정되어야 한다.
* 프로비전 모드를 사용하면 예약 인스턴스를 통해 비용을 절감할 수 있다.

<figure><img src="../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>



#### 스냅샷, DR

* 특정 클러스터 유형에 대해 멀티 AZ모드를 제공한다.
* 일반적인 클러스터 유형에서는 싱글 AZ를 사용하는데, 이 경우에는 DR를 위해서 스냅샷을 주기적으로 내보내주어야 한다.
* 스냅샷은 클러스터를 위한 point-in-time 백업이며 Amazon S3에 내부적으로 저장되고, 계속해서 새로운 스냅샷을 저장하는 것이 아니라 변경된 사항들만 덧붙여진다.
* 새로운 클러스터에 스냅샷을 적용할 수 있다.
* 두 가지 모드가 존재한다.
  * 수동 모드
    * 직접 스냅샷을 내보내야 한다.
    * 스냅샷의 저장 기간이 따로 없으며, 직접 제거할 때 까지 유지된다.
  * 자동 모드
    * 스냅샷을 내보내는 주기를 정해 일정 간격으로 내보낼 수 있다.
    * 스냅샷을 위한 저장 기간을 설정할 수 있다. 1 \~ 35일 동안 유지할 수 있다.
* 스냅샷을 다른 AWS 리전에 자동으로 복사하도록 구성할 수 있다.

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

#### 데이터 수집

* Amazon Kinesis Data Firehose
  * 다양한 소스에서 데이터를 받는 Firehose에서 Redshift로 데이터를 전송할 수 있다.
  * 데이터를 먼저 Amazon S3 버킷에 넣은 후 Kinesis Data Firehose가 자동적으로 S3 복사 명령을 내려서해당 데이터를 Redshift로 로드할 수 있다.
* 수동으로 copy 명령을 이용해 S3 버킷의 데이터를 Redshift로 복사
  * IAM 역할을 함께 사용해야 한다.
  * S3 버킷은 퍼블릭 접근이 가능하므로 인터넷을 통해 데이터를 복사하거나, VPC 라우팅을 이용해 내부적으로 데이터를 복사할 수 있다.
* JDBC 드라이버 사용
  * EC2 인스턴스에서 구동된 애플리케이션로부터 데이터를 Redshift 클러스터에 보낼 때 사용한다.
  * 배치로 데이터를 보내는 것이 좋다.

#### Redshift Spectrum

* Amazon S3의 데이터를 Redshift로 로드하지 않으면서 데이터를 분석할 때 사용한다.
* **Redshift 클러스터에 쿼리를 보내면 수천개의 Redshift Spectrum 노드에게 전달된다.**
* Spectrum 노드들은 Amazon S3에서 데이터를 읽고 병합하여 완료된 결과를 Amazon Redshift 클러스터로 전송한다.
* 이 기능은 Redshift의 처리 기능을 훨씬 더 많이 사용할 수 있다.

## OpenSearch

* 데이터베이스에서는 기본 키나 인덱스만을 이용해서 쿼리를 할 수 있지만 OpenSearch를 사용하면 모든 필드를 검색할 수 있다. 부분 매칭도 가능하다.
* OpenSearch는 보통 검색에 사용되지만, 분석적 쿼리에도 사용할 수 있다.
* OpenSearch 클러스터 프로비저닝 모드
  * 관리형 클러스터
    * 실제 물리적인 인스턴스가 프로비저닝된다.
  * 서버리스 클러스터
    * 스케일링부터 운영까지 모두 AWS에서 관리한다.
* 자체적으로 SQL을 지원하진 않지만, 플러그인을 통해서 SQL 호환성을 활성화할 수 있다. 기본적으로는 자체 쿼리 언어를 사용해야 한다.
* Kinesis Data Firehose, AWS IoT, CloudWatch Log, 커스텀 애플리케이션으로부터 데이터를 받을 수 있다.
* Cognito, IAM, KMS 암호화, TLS를 통해 보안이 제공된다.
* OpenSearch 대시보드로 OpenSearch 데이터를 시각화할 수 있다.
* 사용 패턴
  * 실제 데이터를 담는 DynamoDB Table이 있고, DynamoDB Stream과 람다 함수를 통해 OpenSearch에 데이터를 복사해둘 수 있다. 애플리케이션에서는 OpenSearch를 통해 특정한 항목을 검색할 수 있다.
  * CloudWatch Logs를 Subscription Filter와 람다 함수를 사용해 실시간으로 OpenSearch에 복제할 수도 있고, Subscription Filter와 Kinesis Data Firehose를 사용해 거의 실시간으로 OpenSearch에 복제할 수도 있다.
  * Kinesis Data Streams를 Kinesis Data Firehose와 람다 함수를 사용해 거의 실시간으로 OpenSearch에 저장할 수 있다. 이 때 람다 함수에서는 원하는 형태로 데이터를 변환할 수 있다. 혹은 람다 함수만을 이용해 실시간으로 데이터 스트림을 읽어 OpenSearch에 저장할 수 있다.

## EMR

* Elastic MapReduce
* AWS에서 빅 데이터 작업을 위한 하둡 클러스터 생성에 사용된다. 방대한 양의 데이터를 분석하고 처리할 수 있다.
* 하둡 클러스터는 프로비저닝해야 하며 수백 개의 EC2 인스턴스로 구성 가능하다.
* 빅 데이터 전문가가 사용하는 여러 도구 중 설정이 어려운 도구와 쉽게 통합된다. 예를 들어Apache Spark, HBase, Presto Apache Flink를 사용하고자 할 때 Amazon EMR이 프로비저닝과 구성을 대신 처리해준다.
* 전체 클러스터를 자동으로 확장할 수 있고, 스팟 인스턴스와 통합 가능하다.
* 데이터 처리와 기계 학습, 웹 인덱싱, 빅 데이터 작업에 사용될 수 있다.
* 노드 타입
  * 마스터 노드: 클러스터를 관리하고 다른 모든 노드의 상태를 관리한다. 장기적으로 실행되어야 한다.
  * 코어 노드: 태스크를 실행하고 데이터를 저장한다. 장기적으로 실행되어야 한다.
  * 태스크 노드: 테스크를 실행한다. 대부분 일시적인 스팟 인스턴스를 활용한다. 태스크 노드 사용은 선택 사항이다.
* 구매 옵션
  * 온디맨드 EC2 인스턴스 유형을 사용하면 신뢰할 수 있고 예측 가능한 유형의 워크로드를 얻게 된다.
  * 최소 1년을 사용해야 하는 EC2 예약 인스턴스를 사용하여 비용을 절약할 수 있다. 장기 실행해야 하는 마스터 노드와 코어 노드에 적합하다.
  * 언제든 종료될 수 있는 스팟 인스턴스는 신뢰도는 떨어지지만 저렴하여 태스크 노드에 활용하기 좋다.
* 장기 실행 클러스터로 사용할 수도 있고, 임시 클러스터를 사용해 특정 작업을 수행하고 분석 완료 후에 삭제할 수 있다.

## QuickSight

* 서버리스 머신 러닝 기반 비즈니스 인텔리전스 서비스
* 대시보드를 생성하고 소유한 데이터 소스와 연결할 수 있다.
* 빠르고, 오토 스케일링이 가능하다.
* 웹사이트에 임베드할 수 있으며 세션당 비용을 지불해야 한다.
* 비즈니스 분석, 시각화, 시각화된 정보를 통한 임시 분석 수행, 데이터를 활용한 비즈니스 인사이트 획득에 활용할 수 있다.
* SPICE 엔진
  * 인 메모리 연산 엔진이며 Amazon QuickSight로 데이터를 직접 가져올 때 사용된다.
  * Amazon QuickSight가 이미 다른 DB와 연결되어 있을 때는 작동하지 않는다.
* 엔터프라이즈 에디션에서는 액세스 권한이 없는 사용자에게 일부 열이 표시되지 않도록 열 수준 보안(CLS)을 설정할 수 있다.
* 데이터 소스로는 다양하게 통합 가능하다.
  * RDS, Aurora, Athena, Redshift, S3, Opensearch, Timestream 등 다양한 AWS 서비스와 연결할 수 있다.
  * SaaS인 Salesforce와 Jira 등과도 통합 가능하다.
  * Teradata 같은 타사 데이터베이스와 통합 가능하다.
  * 내부적으로 JDBC 프로토콜을 사용하는 온프레미스 데이터베이스와 통합 가능하다.
  * Excel 파일, CSV 파일, JSON 파일, TSV 파일, 로그 형식의 ELF 및 CLF 등의 데이터 소스를 가져올 수 있다.
* 대시보드 및 분석
  * 사용자와 그룹을 정의할 수 있다. 그룹은 엔터프라이즈 버전에만 제공된다. IAM의 사용자와는 다른 개념이다.
  * 대시보드는 읽기 전용 스냅샷이며 분석 결과를 공유할 수 있다. 또한 분석에 대한 설정(필터, 파라미터 등)을 보존한다.
  * 특정 사용자 또는 그룹과 분석 결과나 대시보드를 공유할 수 있다.
  * 액세스 권한이 있는 사용자는 기본 데이터를 볼 수도 있다.

## Glue

* 추출과 변환 로드 서비스를 관리하는 ETL 서비스
* 분석을 위해 데이터를 준비하고 변환한다.
* 완전한 서버리스 서비스이다.
* 예를 들어 S3 버킷이나 Amazon RDS 데이터베이스에 있는 데이터를 데이터 웨어하우스인 Redshift에 로드할 경우, Glue를 사용해 데이터를 추출한 다음 일부 데이터를 필터링하거나열을 추가하는 등 데이터를 변형하여 Redshift 데이터 웨어하우스에 로드할 수 잇다.

<figure><img src="../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

* Athena에서 사용하기 적합한 Parquet 형식으로 변환할 수도 있다. S3에 데이터가 추가되면 람다 함수가 Glue ETL Job을 트리거시켜, S3에서 CSV 파일을 가져온 후 Glue 서비스 내에서 Parquet 형식으로 변환하도록 한다. 이후 다른 S3 버킷으로 데이터를 내보내어 Amazon Athena가 분석하도록 할 수 있다.
* Glue Data Catalog
  * Glue 데이터 크롤러를 실행해 Amazon S3, Amazon RDS, Amazon DynamoDB 또는 호환 가능한 온프레미스 JDBC 데이터베이스에 연결하여 데이터베이스의 테이블 열, 데이터 형식 등의 모든 메타 데이터를 Glue 데이터 카탈로그에 기록할 수 있다.
  * ETL을 수행하기 위한 Glue 작업에 활용될 모든 데이터베이스, 테이블 메타 데이터를 준비할 수 있다.
  * Amazon Athena, Amazon Redshift Spectrum, Amazon EMR는 데이터와 스키마를 검색할 때 백그라운드에서 AWS Glue Data Catalog를 활용한다.
* 다음과 같은 기능들을 추가적으로 제공한다.
  * Glue Job Bookmarks
    * **새 ETL 작업을 실행할 때 이전 데이터의 재처리를 방지한다.**
  * Glue Elastic Views
    * SQL을 사용해 여러 데이터 스토어의 데이터를 결합하고 복제한다.
    * 커스텀 코드를 지원하지 않으며, Glue가 원본 데이터의 변경 사항을 모니터링한다.
  * Glue DataBrew
    * 사전 빌드된 변환을 사용해 데이터를 정리하고 정규화한다.
  * Glue Studio
    * ETL 작업을 생성, 실행 및 모니터링하는 GUI
  * Glue Streaming ETL
    * Apache Spark Structured Streaming 위에 빌드되며 ETL 작업을 배치 작업이 아니라 스트리밍 작업으로 실행할 수 있다.
    * Kinesis Data Streaming Kafka 또는 AWS의 관리형 Kafka인 MSK에서 Glue 스트리밍 ETL을 사용해 데이터를 읽을 수 있다.

## Lake Formation

* 데이터 레이크란 데이터 분석을 위해 모든 데이터를 한곳으로 모아 주는 중앙 집중식 저장소이다.
* Lake Formation은 데이터 레이크 생성을 수월하게 해 주는 완전 관리형 서비스이다.
* 보통 수개월씩 걸리는 데이터 처리 작업을 며칠 만에 완료할 수 있다.
* 데이터 레이크에서 데이터 검색, 정제, 변환, 추출이 가능하다.
* 여러 작업들을 자동화한다. 이를 통해 데이터 수집, 정제, 카탈로그화, 복제 혹은 기계 학습(ML) 변환 기능으로 중복 제거를 수행할 수 있다.
* 정형 데이터와 비정형 데이터 소스를 결합할 수 있다.
* 블루프린트를 제공하여 데이터를 데이터 레이크로 마이그레이션하는 것을 도와준다. Amazon S3, Amason RDS 온프레미스의 RDB, NoSQL DB 등에서 지원된다.
* 애플리케이션을 연결할 때 **행, 열 수준의 세분화된 액세스 제어**를 할 수도 있다.
* AWS Glue 위에 빌드되는 계층이지만 Glue와 직접 상호 작용하지 않는다.

### 아키텍처

* 데이터 소스로 Amazon S3 RDS, Aurora, SQL, NoSQL 같은 온프레미스 데이터베이스를 사용할 수 있다. 이 때 Lake Formation의 블루프린트를 통해 데이터를 추출(ingest)한다.
* Lake Formation에는 소스 크롤러와 ETL 및 데이터 준비 도구 데이터 카탈로깅 도구가 포함된다. 이는 Glue의 기본 서비스에 해당된다. 데이터를 보호하는 보안 설정과 액세스 제어도 포함된다.
* Athena, Redshift, EMR, Apache Spark 프레임워크 같은 분석 도구가 Lake Formation의 데이터를 사용하게 된다.

### 중앙화된 권한

* 사용자는 허용된 데이터에만 읽기 권한이 있어야 한다.
* Athena와 QuickSight로 데이터 분석을 할 때, S3, RDS, Aurora 같이 각 데이터 소스마다 분석 툴에 대한 권한을 부여하게 되면 보안의 관리 요소가 증가한다.
* Lake Formation에 주입된 모든 데이터는 중앙 S3 버킷에 저장되지만 모든 액세스 제어와 행, 열 수준 보안은 Lake Formation 내에서 관리된다.
* Athena, QuickSight 등 어떤 도구를 사용하든 **Lake Formation에 연결하면 한곳에서 보안을 관리**할 수 있다.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

## Kinesis Data Analytics

### SQL 애플리케이션용 Kinesis Data Analytics

* Kinesis Data Streams와 Kinesis Data Firehose의 데이터를 SQL 문을 기반으로 실시간 분석할 수 있도록 해주는 서비스이다.
* Kinesis Data Streams와 Kinesis Data Firehose 중 하나의 데이터 소스에서 데이터를 읽는다.
* SQL 문을 적용하여 실시간 분석을 처리할 수 있다.
* Amazon S3 버킷의 데이터를 참조해 참조 데이터를 조인할 수도 있다.
* Kinesis Data Streams 또는 Kinesis Data Firehose에 데이터를 전송할 수 있다.
  * Kinesis Data Streams는 Kinesis Data Analytics의 실시간 쿼리로 스트림을 생성하여 AWS Lambda, EC2의 애플리케이션에서 실시간으로 처리하도록 할 수 있다.
  * Kinesis Data Firehose로 바로 전송하는 경우  Amazon S3, Amazon Redshift 혹은 Amazon OpenSearch 등 Firehose 타겟으로 전송된다.
* 특징
  * 완전 관리형 서비스이므로 서버를 프로비저닝하지 않는다.
  * 오토 스케일링이 가능하다.
  * Kinesis Data Analytics에 전송된 데이터만큼 비용을 지불한다.
  * 주로 시계열 분석과 실시간 대시보드와 실시간 지표 용도로 사용된다.

### Apache Flink용 Kinesis Data Analytics

* Apache Flink를 사용하면 Java, Scala, SQL로 애플리케이션을 작성하고 스트리밍 데이터를 처리, 분석할 수 있다.
* Kinesis Data Analytics의 Flink 전용 클러스터에서 Flink 애플리케이션을 백그라운드로 실행할 수 있다.
* Apache Flink을 사용해 두 개의 메인 데이터 소스인 Kinesis Data Streams나 Amazon MSK의 데이터를 읽을 수 있다.
* Apaches Flink는 표준 SQL보다 훨씬 강력하기 때문에 고급 쿼리 능력이나 필요하거나 Kinesis Data Streams나 AWS의 관리형 Kafka인 Amazon MSK 같은 서비스로부터 스트리밍 데이터를 읽는 능력이 필요할 때 사용한다.
* 컴퓨팅 리소스를 자동 프로비저닝할 수 있고 병렬 연산과 오토 스케일링을 할 수 있다.
* 체크포인트와 스냅샷으로 구현되는 애플리케이션 백업이 가능하다.
* Apache Flink는 Kinesis Data Firehose의 데이터는 읽지 못하므로, Kinesis Data Firehose에서 데이터를 읽고 실시간 분석하려면 SQL 애플리케이션용 Kinesis Data Analytics를 사용해야 한다.

## MSK

* AWS의 완전 관리형 Kafka 클러스터 서비스
* MSK 자체적으로 카프카 브로커 노드와 Zookeeper 노드를 생성 및 관리한다.
* 고가용성을 위해 VPC의 클러스터를 최대 세 개의 다중 AZ 전역에 배포한다.
* 일반적인 Kafka 장애를 자동 복구한다.
* EBS 볼륨에 데이터를 저장할 수 있다. 비용만 지불하면 원하는 기간 만큼 계속 저장 가능하다.
* Amazon MSK 서버리스
  * MSK에서 Apache Kafka를 실행하지만 서버 프로비저닝이나 용량 관리가 필요 없다.
  * MSK가 리소스를 자동으로 프로비저닝하고 컴퓨팅과 스토리지를 스케일링한다.
* Apache Kafka는 데이터를 스트리밍하는 방식이다. Kafka 클러스터는 여러 브로커로 구성되고 데이터를 생산하는 생산자는 Kinesis, IoT, RDS 등의 데이터를 클러스터에 주입한다.
* Kafka 토픽으로 데이터를 전송하면 해당 데이터는 다른 브로커로 복제된다.
* Kafka 토픽은 실시간으로 데이터를 스트리밍하고, 소비자는 데이터를 소비하기 위해 토픽을 폴링한다.
* 소비자
  * 데이터로 원하는 대로 처리하거나 EMR, S3, SageMaker, Kinesis RDS 등의 대상으로 보내 처리할 수 있다.
  * Apache Flink용 Kinesis Data Analytics를 사용해 Apache Flink 앱을 실행하고 MSK 클러스터의 데이터를 읽어 분석할 수 있다.
  * Apache Spark Streaming으로 구동된 AWS Glue로 ETL 작업을 스트리밍할 수 있다.
  * Amazon MSK를 이벤트 소스로 이용해 Lambda 함수가 호출되도록 할 수 있다.
  * 자체 Kafka 소비자를 생성해 EC2, EKS 등에서 실행할 수 있다.
* Kinesis Data Streams와의 차이점
  * 메시지 크기 제한: Kinesis Data Streams는 1MB로 제한되어 있지만 Amazon MSK에서는 1MB이 기본값이고 더 큰 메시지를 가지도록 설정 가능하다.
  * 데이터 파티셔닝: Kinesis Data Streams에선 샤드로 나누어 데이터를 스트리밍한다. 용량 확장/축소 시 샤드 분할/병합 작업이 필요하다. Amazon MSK에선 Kafka 토픽을 통해 파티셔닝한다. 토픽 확장만 가능하며, 파티션을 제거하는 기능은 없다.
  * TLS: Kinesis Data Streams에는 TLS 전송 중 암호화 기능이 있고, Amazon MSK에는 평문과 TLS 전송 중 암호화 기능이 있다.
  * 저장된 데이터 암호화: 두 클러스터 모두 가능하다.

## 빅데이터 수집 파이프라인

* 빅데이터 수집 파이프라인의 예시는 다음과 같다.

<figure><img src="../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

* IoT 디바이스들이 실시간으로 데이터를 생산한다. **Amazon Cloud Services의 IoT Core를 통해 데이터를 수집하고 IoT 디바이스를 관리할 수 있다.**
* IoT Core는 데이터를 Kinesis Data Stream으로 전송한다. 빅데이터를 Kinesis 서비스로 실시간으로 파이프라이닝 할 수 있다.
* Kinesis Data Stream은 Kinesis Data Firehose에 데이터를 전송한다.
* Kinesis Data Firehose는 정해진 주기마다 Amazon S3 ingestion(수집) 버킷으로 데이터를 업로드할 수 있다. 이 과정에서 람다 함수를 이용해 데이터를 정리하거나 매우 빠르게 변환할 수 있다.
* 수집 버킷을 통해 SQS 큐 혹은 람다를 작동시킬 수 있다.
* Lambda는 Amazon Athena의 SQL 쿼리를 작동시켜 수집 버킷에서 데이터를 꺼내고 SQL 쿼리를 서버리스로 수행시킨다.
* 쿼리 결과는 S3의 reporting(보고) 버킷으로 전달되어 QuickSight를 통해서 바로 시각화할 수도 있고, Redshift 같은 데이터 웨어하우스로 로드하여 추가적인 분석을 거친 후 QuickSight를 통해 시각화할 수 있다.
