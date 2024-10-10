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

* 소스 애플리케이션 또는 파일로부터 **데이터를 가져와 토픽에 적재**하는 역할을 한다.
* 커스텀 소스 커넥터를 사용하려면 SourceConnector, SourceTask 클래스를 통해 구현하고 jar 파일로 만들어 실행 시 플러그인으로 추가해야 한다.

### 구현해보기

#### 의존성 추가

* connect-api 라이브러리를 추가한다.

```gradle
implementation 'org.apache.kafka:connect-api'
```

#### Config

* AbstractConfig 클래스를 구현하여 커넥터 실행 시 필요한 설정 값들을 정의할 수 있다.
  * ConfigDef 클래스를 생성해 여러 옵션값을 세밀하게 지정할 수 있다.
  * 생성자에서는 상위 클래스 생성자에 설정값들을 담아 호출한다.

```java
public class SingleFileSourceConnectorConfig extends AbstractConfig {

    public static final String DIR_FILE_NAME = "file";
    private static final String DIR_FILE_NAME_DEFAULT_VALUE = "/tmp/kafka.txt";
    private static final String DIR_FILE_NAME_DOC = "읽을 파일 경로와 이름";

    public static final String TOPIC_NAME = "topic";
    private static final String TOPIC_DEFAULT_VALUE = "test";
    private static final String TOPIC_DOC = "보낼 토픽명";

    public static ConfigDef CONFIG = new ConfigDef().define(DIR_FILE_NAME,
                                                    Type.STRING,
                                                    DIR_FILE_NAME_DEFAULT_VALUE,
                                                    Importance.HIGH,
                                                    DIR_FILE_NAME_DOC)
                                                    .define(TOPIC_NAME,
                                                            Type.STRING,
                                                            TOPIC_DEFAULT_VALUE,
                                                            Importance.HIGH,
                                                            TOPIC_DOC);

    public SingleFileSourceConnectorConfig(Map<String, String> props) {
        super(CONFIG, props);
    }
}
```

#### SourceConnector

* SourceConnector는 태스크 실행 전 커넥터 설정 파일을 초기화하고 어떤 태스크 클래스를 사용할 것인지 정의할 때 사용한다.
* 아래는 SourceConnector 구현체 예시이며 하나의 파일을 소스로 사용하여 토픽에 저장하기 위한 커넥터이다.
  * version 메서드는 커넥터의 버전을 반환한다.
  * start 메서드는 입력된 속성을 통해 설정값을 초기화한다. 만약 올바른 값이 입력되지 않았다면 ConnectException을 발생시킨다.
  * taskClass 메서드는 이 커넥터에서 사용할 태스크 클래스를 지정한다.
  * taskConfigs 메서드는 태스크가 여러 개 일때 태스크마다 각기 다른 옵션을 적용하도록 한다.
  * config 메서드는 커넥터가 사용할 설정값 정보를 ConfigDef 타입으로 반환한다.
  * stop 메서드는 커넥터가 종료될 때 호출되므로 자원을 해제하는 등의 작업을 수행하도록 한다.

```java
public class SingleFileSourceConnector extends SourceConnector {

    private final Logger logger = LoggerFactory.getLogger(SingleFileSourceConnector.class);

    private Map<String, String> configProperties;

    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) {
        this.configProperties = props;
        try {
            new SingleFileSourceConnectorConfig(props);
        } catch (ConfigException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public Class<? extends Task> taskClass() {
        return SingleFileSourceTask.class;
    }

    @Override
    public List<Map<String, String>> taskConfigs(int maxTasks) {
        List<Map<String, String>> taskConfigs = new ArrayList<>();
        Map<String, String> taskProps = new HashMap<>();
        taskProps.putAll(configProperties);
        for (int i = 0; i < maxTasks; i++) {
            taskConfigs.add(taskProps);
        }
        return taskConfigs;
    }

    @Override
    public ConfigDef config() {
        return SingleFileSourceConnectorConfig.CONFIG;
    }

    @Override
    public void stop() {
    }
}
```

#### SourceTask

* SourceTask는 소스 애플리케이션 혹은 파일로부터 데이터를 가져와 토픽으로 보낸다.&#x20;
* 자체적으로 소스 애플리케이션 혹은 파일을 어디까지 읽었는지에 대한 오프셋을 가진다. 이를 통해 토픽에 중복으로 데이터를 보내는 것을 방지한다.
* 아래는 SourceTask의 구현체이다.&#x20;
  * version 메서드에서는 태스크의 버전을 지정한다. 보통 커넥터의 버전과 동일하게 둔다.
  * start 메서드에서는 태스크 시작 시 필요한 로직을 작성한다. 데이터 처리에 대한 모든 리소스를 여기서 초기화하면 좋다.
  * poll 메서드에서는 소스 애플리케이션 또는 파일로부터 데이터를 읽어오는 로직을 작성한다. 데이터를 토픽으로 내보내기 위해 SourceRecord 타입으로 데이터를 반환해야 한다.
  * stop 메서드에서는 태스크 종료 시 필요한 로직을 작성한다.

```java
public class SingleFileSourceTask extends SourceTask {
    private Logger logger = LoggerFactory.getLogger(SingleFileSourceTask.class);

    public final String FILENAME_FIELD = "filename";
    public final String POSITION_FIELD = "position";

    private Map<String, String> fileNamePartition;
    private Map<String, Object> offset;
    private String topic;
    private String file;
    private long position = -1;


    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) {
        try {
            // Init variables
            SingleFileSourceConnectorConfig config = new SingleFileSourceConnectorConfig(props);
            topic = config.getString(SingleFileSourceConnectorConfig.TOPIC_NAME);
            file = config.getString(SingleFileSourceConnectorConfig.DIR_FILE_NAME);
            fileNamePartition = Collections.singletonMap(FILENAME_FIELD, file);
            offset = context.offsetStorageReader().offset(fileNamePartition);

            // Get file offset from offsetStorageReader
            if (offset != null) {
                Object lastReadFileOffset = offset.get(POSITION_FIELD);
                if (lastReadFileOffset != null) {
                    position = (Long) lastReadFileOffset;
                }
            } else {
                position = 0;
            }

        } catch (Exception e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public List<SourceRecord> poll() {
        List<SourceRecord> results = new ArrayList<>();
        try {
            Thread.sleep(1000);

            List<String> lines = getLines(position);

            if (lines.size() > 0) {
                lines.forEach(line -> {
                    Map<String, Long> sourceOffset = Collections.singletonMap(POSITION_FIELD, ++position);
                    SourceRecord sourceRecord = new SourceRecord(fileNamePartition, sourceOffset, topic, Schema.STRING_SCHEMA, line);
                    results.add(sourceRecord);
                });
            }
            return results;
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
            throw new ConnectException(e.getMessage(), e);
        }
    }

    private List<String> getLines(long readLine) throws Exception {
        BufferedReader reader = Files.newBufferedReader(Paths.get(file));
        return reader.lines().skip(readLine).collect(Collectors.toList());
    }

    @Override
    public void stop() {
    }
}
```

## 싱크 커넥터

* 토픽의 데이터를 타깃 애플리케이션이나 파일에 저장한다.
* SinkConnector와 SinkTask 클래스를 사용해 싱크 커넥터를 구현할 수 있으며 jar 파일로 만들어 플러그인으로 추가해 사용할 수 있다.

### 구현해보기

#### 의존성 추가

* connect-api 라이브러리를 추가한다.

```gradle
implementation 'org.apache.kafka:connect-api'
```

#### Config

* 토픽의 데이터를 저장할 대상의 정보를 담는다.

```java
public class SingleFileSinkConnectorConfig extends AbstractConfig {

    public static final String DIR_FILE_NAME = "file";
    private static final String DIR_FILE_NAME_DEFAULT_VALUE = "/tmp/kafka.txt";
    private static final String DIR_FILE_NAME_DOC = "저장할 디렉토리와 파일 이름";

    public static ConfigDef CONFIG = new ConfigDef().define(DIR_FILE_NAME,
                                                    Type.STRING,
                                                    DIR_FILE_NAME_DEFAULT_VALUE,
                                                    Importance.HIGH,
                                                    DIR_FILE_NAME_DOC);

    public SingleFileSinkConnectorConfig(Map<String, String> props) {
        super(CONFIG, props);
    }
}
```

#### SinkConnector

* 태스크 실행 전 사용자로부터 입력받은 설정값을 초기화하고 어떤 태스크 클래스를 사용할 지 정의한다.
* 아래는 SinkConnector 구현체이다. 메서드들의 역할은 SourceConnector와 동일하다.

```java
public class SingleFileSinkConnector extends SinkConnector {

    private Map<String, String> configProperties;

    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) {
        this.configProperties = props;
        try {
            new SingleFileSinkConnectorConfig(props);
        } catch (ConfigException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public Class<? extends Task> taskClass() {
        return SingleFileSinkTask.class;
    }

    @Override
    public List<Map<String, String>> taskConfigs(int maxTasks) {
        List<Map<String, String>> taskConfigs = new ArrayList<>();
        Map<String, String> taskProps = new HashMap<>();
        taskProps.putAll(configProperties);
        for (int i = 0; i < maxTasks; i++) {
            taskConfigs.add(taskProps);
        }
        return taskConfigs;
    }

    @Override
    public ConfigDef config() {
        return SingleFileSinkConnectorConfig.CONFIG;
    }

    @Override
    public void stop() {
    }
}
```

#### SinkTask

* 토픽으로부터 데이터를 가져와 애플리케이션이나 파일에 저장하는 로직이 담긴다.
* 아래는 SinkTask의 구현체이다.&#x20;
  * version 메서드에서는 태스크의 버전을 지정한다. 보통 커넥터의 버전과 동일하게 둔다.
  * start 메서드에서는 태스크 시작 시 필요한 로직을 작성한다. 데이터 처리에 대한 모든 리소스를 여기서 초기화하면 좋다.
  * put 메서드에서는 토픽 데이터를 타겟 애플리케이션 또는 파일로 저장하는 로직을 작성한다. SinkRecord는 토픽의 레코드이며, 토픽, 파티션, 타임스탬프 정보를 담고 있다.
  * put 메서드에서는 데이터를 insert하고, flush 메서드에서 커밋하도록 한다면 트랜잭션 형태로 관리할 수 있다.
  * stop 메서드에서는 태스크 종료 시 필요한 로직을 작성한다.

```java
public class SingleFileSinkTask extends SinkTask {
    private SingleFileSinkConnectorConfig config;
    private File file;
    private FileWriter fileWriter;

    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) {
        try {
            config = new SingleFileSinkConnectorConfig(props);
            file = new File(config.getString(config.DIR_FILE_NAME));
            fileWriter = new FileWriter(file, true);
        } catch (Exception e) {
            throw new ConnectException(e.getMessage(), e);
        }

    }

    @Override
    public void put(Collection<SinkRecord> records) {
        try {
            for (SinkRecord record : records) {
                fileWriter.write(record.value().toString() + "\n");
            }
        } catch (IOException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public void flush(Map<TopicPartition, OffsetAndMetadata> offsets) {
        try {
            fileWriter.flush();
        } catch (IOException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public void stop() {
        try {
            fileWriter.close();
        } catch (IOException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }
}
```

