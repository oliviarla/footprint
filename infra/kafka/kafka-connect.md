# Kafka Connect

### 개념

* 반복적인 파이프라인 생성 작업이 필요할 때 프로듀서, 컨슈머 애플리케이션을 개발하고 배포하고 운영하는 것은 비효율적이다. 커넥트를 이용하면 특정 작업 형태를 템플릿으로 만들어 둔 커넥터를 실행하여 반복 작업을 줄일 수 있다.
* 파일의 데이터를 토픽으로 보내는 커넥터는 파일이 존재하는 디렉토리 위치, 파일 이름 등 고유한 설정값을 입력받은 후 실행할 수 있을 것이다.
* 소스 커넥터는 프로듀서 역할을 하며, 싱크 커넥터는 컨슈머 역할을 한다.
* MySQL, S3, MongoDB 등과 같은 저장소가 싱크/소스 애플리케이션에 해당한다.

<figure><img src="../../.gitbook/assets/image (135).png" alt=""><figcaption><p><a href="https://medium.com/walmartglobaltech/kafka-connect-overview-a84782d96ab5">https://medium.com/walmartglobaltech/kafka-connect-overview-a84782d96ab5</a></p></figcaption></figure>

* 커넥트에 커넥터 생성을 요청하면 내부적으로 커넥터와 태스크가 생성된다. 태스크는 커넥터에 종속되어 실질적인 데이터를 처리한다.
* 커넥터를 사용한 파이프라인에 컨버터, 트랜스폼 기능을 추가할 수 있다.
  * 컨버터는 데이터 처리 전 스키마를 변경하도록 도와준다. 예를 들어 JsonConverter, ByteArrayConverter 등이 제공된다.
  * 트랜스폼은 데이터 처리 시 메시지 단위로 데이터를 간단히 변환하려는 용도로 사용된다. 예를 들어 JSON 데이터에서 일부 키 값을 제거/추가할 수 있다.

### 실행 모드

#### 단일 모드 커넥트

* 커넥터를 정의하는 파일을 작성하고 해당 파일을 참조하는 단일 커넥트를 실행하여 파이프라인을 생성할 수 있다.
* 단일 애플리케이션으로 실행되므로 SPOF(Single Point of Failure) 지점이 될 수 있어 고가용성 구성이 불가능하다.
* 주로 개발환경이나 중요도가 낮은 파이프라인 운영 시 사용한다.
* connect-standalone.properties 파일에 커넥트 속성을 설정하여 실행시킬 수 있다.
  * bootstrap.servers : 커넥트와 연동할 카프카 클러스터 주소를 입력한다. 2개 이상 브로커로 이뤄졌다면 `,` 로 구분해 입력한다.
  * key.converter / value.converter : 데이터를 카프카에 저장하거나 카프카에서 가져올 때 사용하는 컨버터 지정
  * key.converter.schemas.enable / value.converter.schemas.enable : 스키마 형태를 사용하는지 여부
  * offset.storage.file.filename : 오프셋을 저장할 로컬 파일의 경로와 이름 지정
  * offset.flush.intervals.ms : 태스크가 처리 완료한 오프셋을 커밋하는 주기 지정
  * plugin.path : 플러그인 형태로 추가할 커넥터의 디렉토리 주소 지정
* connect-file-source.properties 파일에 커넥터 속성을 설정할 수 있다.
  * name : 커넥터 이름을 지정하며, 커넥트 내에서 중복되면 안된다.
  * connector.class : 사용할 커넥터 클래스 이름을 지정한다.
  * tasks.max : 커넥터로 실행할 테스크 개수를 지정한다. 태스크 개수를 늘려 병렬로 처리되도록 할 수 있다.
  * 만약 FileStreamSource 커넥터를 사용한다면 읽을 파일 위치, 데이터를 저장할 토픽의 이름을 지정하는 등, 커넥터마다 필요한 속성을 설정해주어야 한다.
* 아래와 같은 명령으로 커넥트 프로세스를 실행시킬 수 있다.

```bash
bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties
```

#### 분산 모드 커넥트

* 두 대 이상의 서버에서 실행된 프로세스들을 클러스터 형태로 관리해 단일 모드보다 안전하게 운영할 수 있다.
* 데이터 처리량 변화가 있다면 스케일 아웃할 수 있다.
* 커넥터 실행 시 같은 그룹으로 지정된 여러 커넥트들에 분산되어 실행된다. 커넥트 중 한 대에 문제가 생겨도 나머지 커넥트에서 커넥터를 안전히 실행할 수 있다.
* 소스 커넥터, 싱크 커넥터는 데이터 처리 시점을 저장하기 위해 오프셋 정보를 사용한다. 분산 모드 커넥트 이용 시 오프셋 정보는 카프카 내부 토픽에 저장된다. 유실을 막기 위해 복제 개수를 지정할 수 있다.
* connect-distributed.properties 파일에 커넥트 속성을 설정하여 실행시킬 수 있다.
  * bootstrap.servers : 커넥트와 연동할 카프카 클러스터 주소를 입력한다.
  * group.id : 커넥터 프로세서 그룹의 이름을 지정한다.
  * key.converter / value.converter : 데이터를 카프카에 저장하거나 카프카에서 가져올 때 사용하는 컨버터 지정
  * key.converter.schemas.enable / value.converter.schemas.enable : 스키마 형태를 사용하는지 여부
  * offset.storage.topic / offset.storage.replication.factor : 오프셋 저장 토픽 이름, 복제 개수
  * config.storage.topic / config.storage.replication.factor
  * status.storage.topic / status.storage.replication.factor
  * offset.flush.intervals.ms : 태스크가 처리 완료한 오프셋을 커밋하는 주기 지정
  * plugin.path : 플러그인 형태로 추가할 커넥터의 디렉토리 주소 지정
* 분산 모드 커넥트 실행 시 커넥트 설정 파일만 있으면 된다. 커넥터는 커넥트 실행 후 REST API를 통해 실행/중단/변경한다.

```bash
bin/connect-distributed.sh config/connect-distributed.properties
```

## 소스 커넥터

* 소스 애플리케이션 또는 파일로부터 데이터를 가져와 토픽에 적재하는 역할을 한다.
* 커스텀 소스 커넥터를 사용하려면 SourceConnector, SourceTask 클래스를 통해 구현하고 jar 파일로 만들어 실행 시 플러그인으로 추가해야 한다.





## 싱크 커넥터



