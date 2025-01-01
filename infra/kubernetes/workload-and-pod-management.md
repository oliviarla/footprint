# Workload & Pod management

## 워크로드 배치

* 워크로드를 여러 노드에 고르게 분산시켜야 애플리케이션의 가용성을 높일 수 있다.
* 스케줄러가 새로운 파드를 어느 노드에서 실행할 지 결정하는데, 서버의 총 컴퓨팅 파워, 기존 파드의 사용량, 사용자가 파드 실행 위치를 제어하고자 정의한 policy 등에 의해 결정된다.

### 워크로드 배치 상태

* 새로 생성된 파드는 실행될 노드가 지정될 때 까지 pending 상태이다.
* 스케줄러는 파드에 적합한 노드를 지정하기 위해 부적격한 노드를 후보에서 제거하는 filtering 과정과 남은 노드 중 가장 적합한 노드를 선택하기 위한 scoring 과정을 수행한다.

### taint

* 필터링 과정에서는 taint라는 특별한 타입의 레이블이 사용된다. key-value 형태이며 스케줄러가 노드를 분류하는 기준이 된다.
* 컨트롤플레인 노드의 경우 master taint가 부여되어 필터링 과정에서 바로 제외된다.
* 다음 명령들을 이용해 각 노드에 적용된 테인트를 확인하고, 모든 노드에 테인트를 하나 추가하고 제거할 수 있다.

```bash
kubectl get nodes -o jsonpath='{range.items[*]}{.metadata.name} {.spec.taints[*].key}{end}'

# 모든 노드에 taint 추가
kubectl taint nodes --all kiamol-diskk=hdd:Noschedule

# 모든 노드에 해당 taint 제거 (- 기호를 사용)
kubectl taint nodes --all kiamol-diskk=hdd:Noschedule-
```

* 테인트를 추가하더라도 기존 워크로드에는 영향을 미치지 않는다.
* 파드 정의에 toleration을 추가하면 특정 테인트를 가진 노드에도 파드가 배정될 수 있도록 명시할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep2
  labels:
    kiamol: ch19
spec:
  selector:
    matchLabels:
      app: sleep2
  template:
    metadata:
      labels:
        app: sleep2
    spec:
      containers:
      - name: sleep
        image: kiamol/ch03-sleep      
<strong>      tolerations:
</strong><strong>      - key: "kiamol-disk"
</strong><strong>        operator: "Equal"
</strong><strong>        value: "hdd"
</strong><strong>        effect: "NoSchedule"
</strong></code></pre>

* NoSchedule taint가 부여된 노드는 스케줄러의 필터링 과정에서 제외되기 때문에 파드가 배치되지 못한다.
* 파드가 배치되지 못해 pending 상태에 있다면 스케줄러는 계속해서 노드 배정을 시도한다.

#### effect 종류

* NoSchedule
  * 파드 정의에 toleration이 없는 한 스케줄러의 filtering 단계에서 노드가 배제된다.
* PreferNoSchedule
  * 파드 정의에 toleration이 있고 다른 적합한 노드가 남아있지 않을 경우에만 이 노드에서 파드를 실행하게 된다.
  * 스케줄러의 scoring에서 낮은 점수를 받는다.

### 노드셀렉터

* 파드가 특정 레이블을 가진 노드에서만 배치되도록 정의할 수 있다.
* 다음은 노드의 CPU 아키텍처가 ZX 스펙트럼이어야 파드가 배치되도록 노드셀렉터를 이용해 강제한 예시이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep2
  labels:
    kiamol: ch19
spec:
  selector:
    matchLabels:
      app: sleep2
  template:
    metadata:
      labels:
        app: sleep2
    spec:
      containers:
      - name: sleep
        image: kiamol/ch03-sleep
      nodeSelector:        
        kubernetes.io/arch: zxSpectrum
```

## 어피니티

### 노드 어피니티

* 스케줄러에 원하는 우선 조건 또는 필요 조건을 상세히 정의할 수 있다.
* 이를 통해 특정 노드에 파드가 실행되도록 강제할 수 있다.
* requiredDuringSchedulingIgnoredDuringExecution 키워드를 통해 반드시 충족되어야 하는 조건을 지정할 수 있다.
* preferredDuringSchedulingIgnoredDuringExecution 키워드를 통해 더 선호하는 조건을 지정할 수 있다.
* 여러 matchExpressions를 지정하여 원하는 레이블 조건을 명시할 수 있으며, 각 조건은 OR 조건으로 연결된다.
* 아래는 requiredDuringSchedulingIgnoredDuringExecution 키워드를 이용해 파드가 배치될 노드가 운영체제가 linux 혹은 windows여야만 하고, preferredDuringSchedulingIgnoredDuringExecution 키워드를 이용해 리눅스 노드를 좀 더 선호하도록 어피니티를 지정한 예제이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep2
  labels:
    kiamol: ch19
spec:
  selector:
    matchLabels:
      app: sleep2
  template:
    metadata:
      labels:
        app: sleep2
    spec:
      containers:
      - name: sleep
        image: kiamol/ch03-sleep      
      tolerations:
      - key: "kiamol-disk"
        operator: "Equal"
        value: "hdd"
        effect: "NoSchedule"
      affinity:
<strong>        nodeAffinity:
</strong><strong>          requiredDuringSchedulingIgnoredDuringExecution:
</strong>            nodeSelectorTerms:
              - matchExpressions:
                - key: kubernetes.io/arch
                  operator: In
                  values:
                  - amd64
                - key: kubernetes.io/os
                  operator: In
                  values:
                  - linux
                  - windows
              - matchExpressions:
                - key: beta.kubernetes.io/arch
                  operator: In
                  values:
                  - amd64
                - key: beta.kubernetes.io/os
                  operator: In
                  values:
                  - linux
                  - windows
<strong>          preferredDuringSchedulingIgnoredDuringExecution:
</strong>          - weight: 1
            preference:
              matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
          - weight: 1
            preference:
              matchExpressions:
              - key: beta.kubernetes.io/os
                operator: In
                values:
                - linux
</code></pre>

### 파드 어피니티

* 파드 간 노드 배정 시, 다른 파드와 같은 노드에 배정해야 할 때는 어피니티를 작성하고 다른 노드에 배정해야할 때는 안티어피니티를 작성한다.
* 서로 통신하는 컴포넌트들은 같은 노드에 두어 네트워크 부하를 줄일 수 있으며, 같은 컴포넌트들은 서로 다른 노드에 두어 고가용성을 확보할 수 있다.
* 스케줄러가 파드 어피니티 조건에 맞는 노드를 찾아 배정하지 못하면 파드는 pending 상태가 된다.
* topologykey를 이용해 어피니티의 적용 단위를 결정한다. hostname일 경우 파드를 같은 노드에 실행하라는 의미이고, region, zone일 경우 지역이 같은 노드끼리 실행하라는 의미가 된다.
* 아래는 새로 배치할 파드가 app=numbers 레이블이 부여된 파드가 실행중인 노드에 배정하도록 강제하는 예시이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment  
metadata:
  name: numbers-web
  labels:
    kiamol: ch19
    app: numbers
spec:
  selector:
    matchLabels:
      app: numbers
      component: web
  template:
    metadata:
      labels:
        app: numbers
        component: web
    spec:
      containers:
        - name: web
          image: kiamol/ch03-numbers-web
<strong>      affinity:
</strong><strong>        podAffinity:
</strong><strong>          requiredDuringSchedulingIgnoredDuringExecution:
</strong><strong>          - labelSelector:
</strong>              matchExpressions:
              - key: app
                operator: In
                values:
                - numbers
              - key: component
                operator: In
                values:
                - api
            topologyKey: "kubernetes.io/hostname"
</code></pre>

* 파드 어피니티는 노드 어피니티와 같은 규칙을 사용하므로 두 가지를 섞어 사용하면 혼동될 수 있다.
* 디플로이먼트 혹은 레플리카셋 등 컨트롤러에서 파드 안티어피니티를 지정했을 때 조건에 맞는 노드가 더이상 없는 경우 레플리카수를 늘리긴 하지만 늘어난 파드들의 상태는 pending으로 남아있게 된다.

## 자동 스케일링

* 애플리케이션의 부하에 맞춰 레플리카 수를 조절하는 기능이다.
* 기존 파드의 부하를 확인하기 위해 metrics-server 컴포넌트가 제공하는 `kubectl top nodes` 명령을 사용해 리소스 사용량을 확인할 수 있다.
* 쿠버네티스에서는 자동 스케일링을 위해 클러스터 측정값을 수집해야 하므로 metrics-server 컴포넌트가 반드시 설치되어 있어야 한다.

### HPA

* HorizontalPodAutoscaler의 약자로, 자동 스케일링을 위한 리소스를 의미한다.
* 디플로이먼트나 스테이트풀셋 같은 컨트롤러를 스케일링 대상으로 삼으며, CPU 사용량에 따라 조절할 레플리카 수의 범위를 지정할 수 있다.
* 실행중인 모든 파드의 평균 CPU 사용량이 증가하면 15초에 하나씩 파드 개수를 늘리고, 감소하면 5분 이상 CPU 사용량 기준 이하를 유지하는지 확인한 후 파드 개수를 줄인다.
* 측정값이 수집되어 HPA에 전달 되는 데에 최대 수십초의 시간이 걸릴 수 있다.
* 하나의 대상만 지정할 수 있으므로 각 애플리케이션마다 적합한 스케일링 규칙 및 속도를 지정할 수 있다.
* 아래는 기준 CPU 사용량을 75%로 두고 해당 사용량을 기준으로 파드 개수를 1\~5개 사이에서 자동으로 조절하도록 정의한 HPA 예시이다.

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: pi-cpu
  labels:
    kiamol: ch19
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pi-web
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 75
```

* HPA 버전 2에서는 측정 값이 기준 이상 혹은 이하임이 일정 시간동안 지속될 때 까지 대기한 후 스케일링하도록 할 수 있다. 그리고 CPU 사용량 외에 HTTP 요청 수나 큐에 쌓인 메시지 개수 등의 기준을 사용할 수도 있다.
* 다음은 CPU 사용률을 기준으로 하며 파드를 감소시킬 때 기준치 이하로 내려오고 30초동안 유지되면 현재 파드 수를 50% 감소시키는 예시이다.

```yaml
apiVersion: autoscaling/v2beta2  
kind: HorizontalPodAutoscaler
metadata:
  name: pi-cpu
  labels:
    kiamol: ch19
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pi-web
  minReplicas: 1
  maxReplicas: 5  
  metrics:
  - type: Resource
    resource:
      name: cpu      
      target: 
        type: Utilization
        averageUtilization: 75
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 30
      policies:
      - type: Percent
        value: 50
        periodSeconds: 15
```

* 아래 명령을 통해 현재 HPA의 상태를 확인할 수 있다.

```
kubectl get hpa <hpa name>
```

## 파드 축출

* 노드의 컴퓨팅 리소스가 고갈된 경우 서버 상태를 안정화시키기 위해 파드를 제거한다.
* 선점이란 리소스 정의나 리소스쿼터, 배치 및 스케일링 중에 문제가 생겨 메모리, 디스크 용량 등이 고갈된 상태가 발생했을 때 일어난다.
  * CPU 리소스의 경우 다른 컴퓨팅 리소스와 달리 스로틀링만으로 리소스 회수가 가능하므로 선점을 일으키지 않는다.
* 선점이 발생하면 쿠버네티스는 해당 노드가 과부하 상태라고 간주하고 파드를 축출(eviction)한다.
* 축출된 파드는 나중에 문제 원인 파악을 위해 노드에 남겨두지만 파드 컨테이너는 종료 및 삭제된다.
* 축출된 파드가 컨트롤러 관리 하에 있었다면, 다른 노드에 다시 배정된다.
* 파드가 축출되면서 대량의 메모리가 회수되고 메모리 고갈 상태가 사라지면, 이를 대체하는 새 파드가 생성되고 다시 이 파드로 인해 메모리 고갈 상태에 빠지게 되어 계속 새로 생성되고 축출되는 상황이 반복될 수 있다.

### 우선순위

* 축출 대상이 될 수 있는 모든 파드의 우선 순위가 같다면, 초과 점유하고 있는 리소스양이 많은 순서대로 파드를 축출한다. 따라서 파드 정의에 지정하는 기준 리소스 사용량을 신중하게 정해야 한다.
* 파드의 우선순위에 따라 축출될 파드가 결정된다. 파드의 우선순위는 파드에 정의된 리소스 사용량 대비 실제 리소스 사용량과 priority class에 의해 결정된다.
* priority class는 다음과 같이 정의할 수 있으며 값이 클수록 우선순위가 높은 것이다. 메모리 고갈 상태가 발생하면 우선순위가 낮은 파드가 먼저 축출된다.
* 다음은 PriorityClass 리소스를 정의하고 이를 파드 정의에 연결시켜 파드의 우선순위를 높이는 예시이다.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: kiamol-high
  labels:
    kiamol: ch19
value: 10000
globalDefault: false
description: "High priority - may evict low priority"
```

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: stress-high
  labels:
    kiamol: ch19
spec:
  replicas: 2
  selector:
    matchLabels:
      app: stress
      level: high
  template:
    metadata:
      labels:
        app: stress
        level: high
    spec:
<strong>      priorityClassName: kiamol-low
</strong><strong>      # ...
</strong></code></pre>
