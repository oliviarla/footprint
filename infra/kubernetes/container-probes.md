# Container Probes

## 컨테이너 프로브

* 자기수복형 애플리케이션이란 클러스터 자체에서 애플리케이션의 일시적인 문제를 찾아 해결할 수 있는 것을 의미한다.
* 파드 컨테이너에서 애플리케이션이 실행중이고 단지 요청이 실패하는 상황이라면 쿠버네티스에서는 파드가 정상이라고 판단할 수밖에 없다. 이러한 상황에서 해당 파드로 들어가는 요청을 막기 위해 컨테이너 프로브를 사용할 수 있다.
* 애플리케이션을 안정적이고 지속적으로 운영하려면 컨테이너 프로브를 통해 애플리케이션 정상 여부를 확인하고, 리소스 부족 상태에 빠지지 않도록 한계를 적절히 설정해야 한다.
* 컨테이너 프로브는 파드 정의에 기술되며, 일정 주기로 실행되어 애플리케이션의 상태가 정상인지 판단한다.

### 레디니스 프로브

* 레디니스 프로브는 네트워크 수준에서 처리된다.
* 파드 컨테이너 상태가 비정상이면, 해당 파드는 READY 상태에서 제외되고 서비스의 활성 파드 목록에서도 제외된다. 따라서 서비스로 들어온 트래픽을 더이상 처리하지 않는다.
* 일부 파드에 일시적으로 과부하가 걸렸을 때 레디니스 프로브에 의해 잠시 서비스에서 제외시키고, 상태가 정상적으로 돌아오면 다시 서비스에 복귀하도록 사용할 수 있다.
* 아래는 /healthz 로 HTTP GET 요청을 보내 애플리케이션의 상태를 5초마다 확인하기 위한 레디니스 프로브 정의이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: numbers-api
  labels:
    kiamol: ch12
spec:
  replicas: 2
  selector:
    matchLabels:
      app: numbers-api
  template:
    metadata:
      labels:
        app: numbers-api
        version: v2
    spec:
      restartPolicy: Always
      containers:
        - name: api
          image: kiamol/ch03-numbers-api
          ports:
            - containerPort: 80
              name: api
          env:
            - name: FailAfterCallCount
              value: "1"
<strong>          readinessProbe:
</strong><strong>            httpGet:
</strong><strong>              path: /healthz
</strong><strong>              port: 80
</strong><strong>            periodSeconds: 5
</strong></code></pre>

* 서비스에 연결된 모든 파드가 응답할 수 없는 상태가 된다면, 서비스는 요청을 더이상 처리하지 못하게 된다. 따라서 애플리케이션이 자동으로 복구되지 않는다면 별도 처리 없이는 영원히 해당 서비스를 사용하지 못할 수 있다.

### 리브니스 프로브

* 리브니스 프로브는 파드 정의에 기술되며, 고장을 일으킨 파드 컨테이너를 재시작한다.
* initialDelaySeconds 속성으로 첫번째 상태 체크 전 대기할 수 있으며, failureThreshold 속성으로 상태 체크 실패를 몇번 용인할 것인지 설정할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: numbers-api
  labels:
    kiamol: ch12
spec:
  replicas: 2
  selector:
    matchLabels:
      app: numbers-api
  template:
    metadata:
      labels:
        app: numbers-api
        version: v3
    spec:
      restartPolicy: Always
      containers:
        - name: api
          image: kiamol/ch03-numbers-api
          ports:
            - containerPort: 80
              name: api
          env:
            - name: FailAfterCallCount
              value: "1"
          readinessProbe:
            httpGet:
              path: /healthz
              port: 80
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /healthz
              port: 80
            periodSeconds: 10
            initialDelaySeconds: 10
            failureThreshold: 2
```

* 애플리케이션이 계속해서 고장을 일으킨다면 계속 재시작하는 데에는 한계가 있으므로 완전한 해결책이 되지 못한다.
* 롤아웃으로 애플리케이션 업데이트를 진행할 때 레디니스 프로브를 통해 상태 체크에 실패하면 롤아웃을 진행하지 않도록 할 수 있어 유용하다.
* CrashLoopBackOff 상태는 정상적으로 실행될 수 없는 파드를 재시작하느라 클러스터 자원이 낭비되는 것을 막아준다. 파드가 재시작되는 시간 간격을 점차 늘려 재시작을 시도하는데, 이 상태에 빠졌다는 것은 스스로 수복할 수 없다는 것을 의미한다.

### 헬름 차트와 통합

* 헬름 차트에서는 `--atomic` 옵션을 통해 작업이 실패할 경우 자동으로 애플리케이션을 롤백해주는데, 프로브의 상태 체크도 작업 실패로 간주한다.
* 헬름 업그레이드 전에 수행할 잡을 설정하여 해당 잡에 테스트나 검증하는 내용을 작성하고 실패 시 업그레이드를 실패하도록 할 수도 있다.
* 헬름 차트에서는 kubectl에서 계속 파드를 재시작하거나 CrashLoopBackOff 상태가 되어버리는 것과 달리 자동으로 애플리케이션을 롤백시켜준다.
