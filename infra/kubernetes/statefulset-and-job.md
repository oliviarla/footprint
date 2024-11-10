# StatefulSet & Job

## 스테이트풀셋

* 도메인 이름으로 식별되는 규칙적인 이름이 부여된 파드를 생성한다. 그리고 파드를 병렬 실행하지 않고 한 파드가 RUNNING이 되면 그다음 파드를 순서대로 생성한다.
* 클러스터 애플리케이션은 스테이트풀셋으로 모델링하기 적합하다. 대부분 클러스터 애플리케이션은 주 인스턴스와 부 인스턴스들이 함께 동작하여 고가용성을 확보하고, 부 인스턴스의 수를 늘려 스케일링한다.
* 레플리카셋은 구조 상 특정 파드를 주 인스턴스로 지정할 수 없다.
* 스테이트풀셋은 상당히 복잡하므로 웬만해서는 사용하지 않는다. 기존 애플리케이션을 쿠버네티스 상으로 마이그레이션하는 경우에는 스테이트풀셋이 유용할 수 있다.
* 아래와 같이 스테이트풀셋을 정의할 수 있다. 가장 먼저 실행된 파드가 주 인스턴스가 될 것이다.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: todo-db
  labels:
    kiamol: ch08
spec:
  selector:
    matchLabels:
      app: todo-db
  serviceName: todo-db
  replicas: 2
  template:
    metadata:
      labels:
        app: todo-db
    spec:
      containers:
        - name: db
          image: postgres:11.6-alpine
          env:
          - name: POSTGRES_PASSWORD_FILE
            value: /secrets/postgres_password
          volumeMounts:
            - name: secret
              mountPath: "/secrets"
      volumes:
        - name: secret
          secret:
            secretName: todo-db-secret
            defaultMode: 0400
            items:
            - key: POSTGRES_PASSWORD
              path: postgres_password
```

* 스테이트풀셋을 실행시키고 확인하는 명령은 아래와 같다.

```bash
# 데이터베이스 스테이트풀셋과 서비스, 비밀값 등을 배치
kubectl apply -f todo-list/db/

# 스테이트풀셋 확인
kubectl get statefulset todo-db

# 파드 확인
kubectl get pods -l app=todo-db
```

* 스테이트풀셋의 파드 이름은 스테이트풀셋 이름 뒤에 번호를 붙여 규칙적으로 부여된다. 따라서 레이블 셀렉터에 의지하지 않고 직접 파드를 관리할 수 있다.
* 파드 수를 감소시키면 뒷 번호부터 차례대로 제거된다. 파드가 고장나더라도 대체 파드는 삭제된 파드 이름과 설정을 그대로 따른다.
* 각 파드를 도메인 이름으로 접근할 수 있어, 클러스터 내 프로세스들이 이미 알고 있는 서로의 주소로 통신할 수 있다.

## 스테이트풀셋에서 초기화 컨테이너 활용

* 초기화 컨테이너는 파드의 실행 순서에 따라 실행된다. 따라서 2개의 파드가 존재하는 스테이트풀셋에서 파드 0의 초기화 컨테이너가 먼저 실행되고, 이후 파드 1의 초기화 컨테이너가 실행될 것이다.
* 복제 클러스터 형태로 동작하는 데이터베이스의 경우, 첫 번째 파드 구동 시 초기화 컨테이너에서 주 인스턴스임을 로깅하고, 두번째 파드부터는 구동 시 초기화 컨테이너에서 주 인스턴스로 접근이 가능한지 확인하고 복제본을 생성하도록 할 수 있다.

<figure><img src="../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

* 아래와 같이 클러스터IP 가 없는 서비스인 헤드리스 서비스를 정의하면 가상 IP 주소 대신 각 파드의 IP 주소를 반환하고, 도메인 네임이 DNS 서버에 등록된다.

```
apiVersion: v1
kind: Service
metadata:
  name: todo-db
  labels:
    kiamol: ch08
spec:
  ports:
    - port: 5432
      targetPort: 5432
      name: postgres
  selector:
    app: todo-db  
  clusterIP: None
```

* 스테이트풀셋은 자신이 관리하는 파드마다 내부 DNS 도메인을 부여하므로, 파드를 주소로 구분할 수 있다.

```bash
# 서비스 이름으로 도메인 조회
kubectl exec deploy/sleep -- sh -c 'nslookup todo-db | grep "^[^*]"'

# 0번 파드에 대한 도메인 조회
kubectl exec deploy/sleep -- sh -c 'nslookup todo-db-0.todo-db.default.svc.cluster.local | grep "^[^*]"'
```

## 볼륨 클레임 템플릿으로 스토리지 요청

* 일반적으로 PVC를 마운트하면 모든 파드가 같은 볼륨에 데이터를 기록할 수 있게 된다.
* 스테이트풀셋 정의에서 volumeClaimTemplates 필드를 정의하면 각 파드마다 PVC가 생성되고 연결된다. 즉, 파드마다 별도 스토리지가 마운트된다.
* 파드가 새로운 파드로 대체되더라도 기존 PVC가 사용된다. 이를 통해 애플리케이션 컨테이너가 새로 생성되더라도 이전과 컨테이너와 동일한 상태를 유지할 수 있다.
* 단, volume claim을 추가하는 등 근본적인 변경이 있으면 기존 스테이트풀셋을 업데이트할 수 없어 아예 스테이트풀셋을 삭제하고 다시 배치해야 한다. 따라서 애플리케이션 요구사항을 신중히 검토하며 설계해야 한다.
* 아래와 같이 volumeClaimTemplates 필드를 정의하면 파드마다 동적으로 PVC가 생성된다. PVC는 기본 스토리지 유형의 PV로 연결된다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: Service
metadata:
  name: sleep-with-pvc
  labels:
    kiamol: ch08
spec:
  selector:
    app: sleep-with-pvc
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sleep-with-pvc
  labels:
    kiamol: ch08
spec:
  selector:
    matchLabels:
      app: sleep-with-pvc
  serviceName: sleep-with-pvc
  replicas: 2
  template:
    metadata:
      labels:
        app: sleep-with-pvc
    spec:
      containers:
        - name: sleep
          image: kiamol/ch03-sleep
          volumeMounts:
            - name: data
              mountPath: /data
<strong>  volumeClaimTemplates:
</strong><strong>  - metadata:
</strong><strong>      name: data
</strong><strong>      labels:
</strong><strong>        kiamol: ch08
</strong><strong>    spec:
</strong><strong>      accessModes: 
</strong><strong>       - ReadWriteOnce
</strong><strong>      resources:
</strong><strong>        requests:
</strong><strong>          storage: 5Mi
</strong></code></pre>

## 잡과 크론잡

### 잡

* **잡**이란 데이터 백업 및 복원 작업에 적합한 파드 컨트롤러이다. 파드 정의를 포함하여 파드가 수행하는 배치 작업이 완료되는 것을 보장한다.
* 반드시 완료되어야 하지만 수행 시점이 언제인지 크게 상관 없는 계산 중심 혹은 입출력 중심 작업을 수행할 때 적합한 리소스이다.
* 클러스터 상에서 실행되어 클러스터 자원을 사용할 수 있다.
* 잡 내부의 파드가 실행하는 프로세스는 작업을 마치고 종료되어야 한다. 종료되지 않는 프로세스를 실행하면 잡은 영원히 완료되지 않을 것이다.
* restartPolicy 필드를 정의하여 배치 작업 실패 시 대응 방침을 지정한다. 파드를 재시작하고 컨테이너만 대체하거나, 파드 자체를 대체할 수도 있다.
* 잡이 성공적으로 완료되면, 종료된 파드는 그대로 유지된다.
* 아래는 원주율 계산 프로세스를 배치 모드로 실행하도록 정의된 잡이다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-job
  labels:
    kiamol: ch08
spec:
  template:
    spec:
      containers:
        - name: pi
          image: kiamol/ch05-pi
          command: ["dotnet", "Pi.Web.dll", "-m", "console", "-dp", "50"]
      restartPolicy: Never
```

```bash
# 잡 배치
kubectl apply -f pi-job.yaml

# 잡 상태 확인
kubectl get job pi-job

# 파드 로그 확인
kubectl get -l job-name=pi-job
```

* 동일한 작업에 여러 입력값을 설정해 파드를 각각 생성해 배치 작업이 클러스터에서 병렬로 진행되도록 할 수 있다. 이 때 아래 두 필드를 지정해야 한다.
  * completions: 잡을 실행할 횟수를 지정한다. 잡은 지정된 수의 파드가 원하는 횟수만큼 작업을 완료하는지 확인한다.
  * parallelism: 동시에 실행할 파드 수를 지정한다. 잡을 수행하는 속도와 작업에 사용할 클러스터의 연산 능력을 조절할 수 있다.
* 아래는 초기화 컨테이너를 이용해 랜덤 값을 파일에 저장해두고, 이를 애플리케이션 컨테이너에서 읽어 원주율 소수점아래 자릿수로 사용하도록 하는 잡이다. 파드 세 개를 동시에 실행하여 작업을 수행할 것이다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-job-random
  labels:
    kiamol: ch08
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      initContainers:
        - name: init-dp
          image: kiamol/ch03-sleep
          command: ['sh', '-c', 'echo $RANDOM > /init/dp']
          volumeMounts:
            - name: init
              mountPath: /init
      containers:
        - name: pi
          image: kiamol/ch05-pi
          command: ['sh', '-c', 'dotnet Pi.Web.dll -m console -dp $(cat /init/dp)']
          volumeMounts:
            - name: init
              mountPath: /init
              readOnly: true
      restartPolicy: Never
      volumes:
        - name: init
          emptyDir: {}
```

### 크론잡

* 잡을 관리하는 컨트롤러로, 주기적으로 잡을 생성한다.
* 잡을 실행할 스케줄은 리눅스 cron 포맷으로 작성한다.
* 작업이 완료된 잡과 파드를 자동으로 삭제하지 않으므로 직접 삭제해주어야 한다.
* 레이블 셀렉터를 통해 관리 대상을 식별하는 방식이 아니므로, 잡 템플릿에 레이블을 추가하지 않았다면 크론잡이 관리하는 잡을 찾기 위해 잡을 하나하나 보며 관리 주체가 해당 크론잡인지 확인해야 한다.
* 크론잡에서 돌아가는 배치 작업 시간이 크론 주기보다 길어지면 다음 주기에 새로운 작업이 또 시작되어 여러 작업이 동시에 수행될 수 있다.
* 다음은 PostgreSQL 데이터베이스로부터 백업 파일을 만들어내 PVC를 통해 연결된 스토리지에 저장하는 정의이다.

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: todo-db-backup
  labels:
    kiamol: ch08
spec:
  schedule: "*/2 * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: backup
            image: postgres:11.6-alpine
            command: ['sh', '-c', 'pg_dump -h $POSTGRES_SECONDARY_FQDN -U postgres -F tar -f "/backup/$(date +%y%m%d-%H%M).tar" todo']
            envFrom:
            - configMapRef:
                name: todo-db-env
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  key: POSTGRES_PASSWORD
                  name: todo-db-secret
            volumeMounts:
              - name: backup
                mountPath: "/backup"
          volumes:
            - name: backup
              persistentVolumeClaim:
                claimName: todo-db-backup-pvc
```

```bash
kubectl apply -f <크론잡 파일>
kubectl get cronjob <크론잡 이름>
```

* suspend를 true로 두어 크론잡을 일시적으로 비활성화하는 보류 모드로 전환할 수 있다.

<pre><code># ...
spec:
  schedule: "*/2 * * * *"
  concurrencyPolicy: Forbid
<strong>  suspend: true
</strong></code></pre>
