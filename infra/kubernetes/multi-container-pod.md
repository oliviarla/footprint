# Multi Container Pod

## 파드와 컨테이너 통신

* 파드는 하나 이상의 컨테이너가 공유할 수 있는 네트워크 및 파일 시스템을 제공하는 가상 환경으로, 한 노드 상에서 동작하는 단위이다.
* 컨테이너는 별도의 환경 변수와 자신만의 프로세스를 가지며 서로 다른 기술 스택으로 구성된 별개 이미지를 사용할 수 있는 독립된 단위이다.
* 윈도우 컨테이너와 리눅스 컨테이너는 같은 노드에서 동작할 수 없다.
* **같은 파드 내에 있는 컨테이너들은 같은 IP 주소를 가진다. 따라서 localhost 통신이 가능**하다.
* 파드에서 제공하는 **볼륨을 마운트하여 공유하면 컨테이너끼리 정보를 교환**할 수 있다.

<figure><img src="../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

* 아래는 디플로이먼트에 멀티컨테이너 파드를 정의한 예제이다. 두 컨테이너를 배열로 정의하였다.
  * 두 컨테이너 사이에 하나의 볼륨을 공유하여 마운트할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
  labels:
    kiamol: ch07
spec:
  selector:
    matchLabels:
      app: sleep
  template:
    metadata:
      labels:
        app: sleep
    spec:
      containers:
        - name: sleep
          image: kiamol/ch03-sleep   
          volumeMounts:
            - name: data
              mountPath: /data-rw    
        - name: file-reader
          image: kiamol/ch03-sleep
          volumeMounts:
            - name: data
              mountPath: /data-ro
              readOnly: true
      volumes:
        - name: data
          emptyDir: {}
```

* 멀티 컨테이너 파드의 로그 출력을 위해서는 특정 컨테이너를 지정해야 한다.

```bash
kubectl logs -l <파드 레이블 조건> -c <컨테이너 이름>
```

## 초기화 컨테이너

* 멀티 컨테이너 파드의 경우 모든 컨테이너 상태가 Ready가 되어야 파드도 준비된 상태가 된다.
* 초기화 컨테이너란 애플리케이션 컨테이너보다 먼저 실행되어 애플리케이션 실행 준비를 돕는 추가적인 컨테이너이다.
* 추가 컨테이너(사이드카)가 애플리케이션 컨테이너(오토바이)를 지원하는 사이드카 컨테이너와는 다르다.
* 초기화 컨테이너는 파드 안에 여러 개 정의할 수 있으며 파드 내에 정의된 순서에 따라 실행된다.
* 모든 초기화 컨테이너가 목표를 달성해야 애플리케이션 컨테이너나 사이드카 컨테이너를 실행한다.
* 초기화 컨테이너의 주요 역할 중 하나는 애플리케이션 컨테이너에서 필요한 환경을 준비하는 것이다.
* initContainers에 초기화 컨테이너를 정의할 수 있다.
  * 아래 정의된 초기화 컨테이너에서는 컨피그맵 볼륨 마운트에서 설정을 읽은 후 환경 변수 설정값을 병합해 emptyDir 볼륨 마운트에 파일로 기록한다.&#x20;
  * 컨테이너는 환경 변수를 공유하지 못하므로 초기화 컨테이너에 지정된 설정 값을 emptyDir 볼륨 마운트에 기록하고, emptyDir 볼륨을 config 디렉토리로 마운트하여 애플리케이션 컨테이너에서 설정 값을 얻어오도록 한다.
  * 만약 컨테이너 이미지에 이미 특정 디렉토리가 포함되어 있고 이를 새로운 볼륨 마운트 공간으로 사용한다면 덮어쓰여지기 때문에 조심해야 한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: timecheck
  labels:
    kiamol: ch07
spec:
  selector:
    matchLabels:
      app: timecheck
  template:
    metadata:
      labels:
        app: timecheck
        version: v2
    spec:
      initContainers:
        - name: init-config
          image: kiamol/ch03-sleep
          command: ['sh', '-c', "cat /config-in/appsettings.json | jq --arg APP_ENV \"$APP_ENVIRONMENT\" '.Application.Environment=$APP_ENV' > /config-out/appsettings.json"]
          env:
          - name: APP_ENVIRONMENT
            value: TEST         
          volumeMounts:
            - name: config-map
              mountPath: /config-in
            - name: config-dir
              mountPath: /config-out
      containers:
        - name: timecheck
          image: kiamol/ch07-timecheck
          volumeMounts:
            - name: config-dir
              mountPath: /config
              readOnly: true
      volumes:
        - name: config-map
          configMap:
            name: timecheck-config
        - name: config-dir
          emptyDir: {}
```

## 어댑터 컨테이너

* 애플리케이션과 컨테이너 플랫폼 사이를 중재하는 어댑터 역할을 맡는 사이드카 컨테이너를 의미한다.
* 도커나 쿠버네티스는 표준 출력 스트림으로 출력된 로그를 컨테이너 로그로 수집한다. 파일에 직접 로그를 남기거나 컨테이너 로그가 수집될 수 없는 채널을 이용해 로그를 남긴다면 파드 로그를 통해 로그를 확인할 수 없다.
* 어댑터 컨테이너를 이용하면 아래와 같이 로그가 출력되는 볼륨을 마운트한 후 해당 로그 파일을 tail -f 명령을 이용해 내용이 추가될 때 마다 출력하도록 해 쿠버네티스 상으로 로그 수집이 가능하도록 할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: timecheck
  labels:
    kiamol: ch07
spec:
  selector:
    matchLabels:
      app: timecheck
  template:
    metadata:
      labels:
        app: timecheck
        version: v3
    spec:
      initContainers:
        # ...
      containers:
        - name: timecheck
          image: kiamol/ch07-timecheck
          volumeMounts:
            - name: config-dir
              mountPath: /config
              readOnly: true
            - name: logs-dir
              mountPath: /logs
<strong>        - name: logger
</strong><strong>          image: kiamol/ch03-sleep
</strong><strong>          command: ['sh', '-c', 'tail -f /logs-ro/timecheck.log'] 
</strong><strong>          volumeMounts:
</strong><strong>            - name: logs-dir
</strong><strong>              mountPath: /logs-ro
</strong><strong>              readOnly: true
</strong>      volumes:
        - name: config-map
          configMap:
            name: timecheck-config
        - name: config-dir
          emptyDir: {}
        - name: logs-dir
          emptyDir: {}
</code></pre>

* 이외에도 커스터마이징된 정보를 수집하거나 헬스 체크, 성능 지표를 수집하는 이미지를 사이드카 컨테이너에서 사용할 수 있다.

```yaml
- name: healthz
  image: kiamol/ch03-sleep  
  command: ['sh', '-c', "while true; do echo -e 'HTTP/1.1 200 OK\nContent-Type: application/json\nContent-Length: 17\n\n{\"status\": \"OK\"}' | nc -l -p 8080; done"]
  ports:
    - containerPort: 8080
      name: http
```

* 애플리케이션 컨테이너에서 외부로 향하는 트래픽을 관리하고 싶다면 어댑터 컨테이너로 프록시 컨테이너를 둘 수 있다.
* 어댑터 역할을 하는 사이드카 컨테이너는 오버헤드로 작용한다. 자원을 더 소모하고 파드 업데이트에 걸리는 시간도 길어질 수 있다.
* 아래 그림과 같이 old 파드에서는 애플리케이션 컨테이너의 한계로 각종 사이드카 컨테이너를 두지만, new 파드에서는 애플리케이션 컨테이너 자체에서 기능을 제공하여 오버헤드를 줄일 수 있다. 단, 두 방식을 외부에서 봤을 때 동작은 완전히 똑같다.

<figure><img src="../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

## 앰배서더 컨테이너

* 앰배서더 컨테이너에서는 애플리케이션과 외부 통신을 제어하고 단순화하여 네트워크의 제어권을 가지게 된다. 이 때 성능 향상 혹은 신뢰성, 보안 강화하는 내용 등 복잡한 로직이 포함될 수 있다.
* 애플리케이션이 네트워크 요청을 보내면 앰배서더 컨테이너가 받아 처리할 수 있다.
* 데이터베이스 앰배서더 컨테이너를 둘 경우 update 쿼리는 마스터 DB에, select 쿼리는 복제 DB에 보내도록 할 수 있다.
* 프록시 컨테이너를 활용하면 서비스 디스커버리, 로드밸런싱, 연결 재시도, 비보안 채널에 대한 암호화 등을 할 수 있게 된다.
* 아래와 같이 프록시 앰배서더 컨테이너를 정의하면 모든 트래픽이 앰배서더 컨테이너를 거치며, 네트워크 요청을 URL 매핑에 따라 라우팅한다. 각 애플리케이션 컨테이너는 호출할 서비스의 URL을 알 필요 없이 프록시의 한 주소로만 요청을 보내면 된다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: numbers-web
  labels:
    kiamol: ch07
spec:
  selector:
    matchLabels:
      app: numbers-web
  template:
    metadata:
      labels:
        app: numbers-web
    spec:
      containers:
        - name: web
          image: kiamol/ch03-numbers-web 
          env:
          - name: http_proxy
            value: http://localhost:1080
          - name: RngApi__Url
            value: http://localhost/api
        - name: proxy
          image: kiamol/ch07-simple-proxy # 요청된 주소를 실제 주소로 매핑하고 로그를 남기는 프로그램
          env:
          - name: Proxy__Port
            value: "1080"
          - name: Proxy__Request__UriMap__Source
            value: http://localhost/api
          - name: Proxy__Request__UriMap__Target
            value: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng
```

## 파드 환경 이해하기

* 파드는 컴퓨팅의 기본 단위이다. 내부에 존재하는 모든 컨테이너가 READY여야 파드도 READY 상태가 된다.&#x20;
* 초기화 컨테이너 실행에 실패하면 애플리케이션 컨테이너도 업데이트되지 않는다. 이 때 새로 생성된 파드는 READY 상태에 들어가지 못하지만, 디플로이먼트는 기존 레플리카셋을 사용하므로 사용 가능 상태이니 주의해야 한다.
* 다음은 파드 및 컨테이너의 재시작 조건이다.
  * 초기화 컨테이너를 가진 파드가 대체될 때, 새 파드는 초기화 컨테이너를 모두 실행하므로 초기화 로직은 반복적으로 실행 가능해야 한다.
  * 초기화 컨테이너 이미지가 변경되면 파드가 재시작된다. 초기화 컨테이너가 다시 실행되며 애플리케이션 컨테이너도 교체된다.
  * 애플리케이션 컨테이너 이미지가 변경되면 초기화 컨테이너는 재시작하지 않고 애플리케이션 컨테이너만 대체된다.
  * 애플리케이션 컨테이너가 종료되면 애플리케이션 컨테이너를 재생성한다.
* 프로세스 간 통신이나 애플리케이션 프로세스의 지표 수집을 위해 사이드카 컨테이너가 애플리케이션 프로세스에 접근해야 할 경우 `shareProcessNamespace: true` 설정을 추가하면 접근이 가능하다.
