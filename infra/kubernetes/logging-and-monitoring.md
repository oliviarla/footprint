# Logging & Monitoring

## 로그 관리 시스템

### 로그 관리 파이프라인

* 각 파드의 컨테이너들로부터 발생한 로그 파일들을 중앙화된 시스템 형태로 관리하기 위해 아래와 같은 파이프라인을 구성할 수 있다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* 컨테이너 로그는 컨테이너가 실행중인 노드에 파일 형태로 저장된다. 로그 파일의 이름에는 네임스페이스, 파드, 컨테이너 이름이 포함된다.
* 컨테이너가 저장한 로그 파일을 로그 수집기(ex. fluentd)가 수집하여 중앙화된 로그 저장소(ex. elastic search)에 전달하면, 운영자는 로그 저장소의 여러 확장 프로그램(ex. kibana)을 이용해 로그 검색 및 필터링 기능을 이용할 수 있다.
* 로그 파일은 파드 컨테이너가 재시작되더라도 그대로 유지된다. 로그 로테이션으로 인해 오래된 로그 파일이 덮어쓰여질 수 있으니 노드에서 로그 파일을 중앙 저장소로 전달하면 더 장기적으로 저장할 수 있다.
* 쿠버네티스 코어 컴포넌트에서 생성된 로그도 동일한 방식으로 수집된다.&#x20;

### 로그 파일 수집

* fluentd를 이용해 각 노드에서 발생한 로그 파일들을 하나의 저장소로 수집할 수 있다.
* 모든 노드에서 fluentd의 경량화 프로세스인 fluent-bit 파드를 데몬셋 형태로 실행시키고, 파드가 로그 파일에 접근할 수 있도록 호스트경로 마운트를 추가해주어야 한다.
* 아래는 fluentd 데몬셋 정의이다.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: kiamol-ch13-logging
  labels:
    kiamol: ch13
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      serviceAccountName: fluent-bit
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:1.8.11
        volumeMounts:
        - name: fluent-bit-config
          mountPath: /fluent-bit/etc/
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: fluent-bit-config
        configMap:
          name: fluent-bit-config
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
---
# RBAC configuration - ignore this until we get to chapter 17 :)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluent-bit
  namespace: kiamol-ch13-logging
  labels:
    kiamol: ch13
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluent-bit
  labels:
    kiamol: ch13
rules:
- apiGroups: [""]
  resources:
  - namespaces
  - pods
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fluent-bit
  labels:
    kiamol: ch13
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: fluent-bit
subjects:
- kind: ServiceAccount
  name: fluent-bit
  namespace: kiamol-ch13-logging
```

* 다음 명령을 이용하면 fluentd가 각각의 컨테이너로부터 읽어들인 로그를 확인할 수 있다.

```
kubectl logs -l app=fluent-bit -n kiamol-ch13-logging --tail 2
```

* ConfigMap의 경우 아래와 같이 작성할 수 있다. 여기서 설정하는 값들을 통해 로그에 다양한 메타데이터를 추가하고 원하는 기준에 따라 서로 다른 대상으로 로그를 출력할 수 있다.
  * fluentd의 데이터 처리는 로그 파일을 읽어들이는 **입력 단계**, JSON 포맷으로 된 원시 로그를 전처리하는 **파싱 단계**, 하나의 엔트리를 fluentd 컨테이너 로그로 **출력하는 단계**로 나뉜다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: kiamol-ch13-logging
  labels:
    kiamol: ch13
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     error
        Daemon        off
        Parsers_File  parsers.conf

    @INCLUDE input.conf
    @INCLUDE filter.conf
    @INCLUDE output.conf

  input.conf: |
    [INPUT]
        Name              tail
        Tag               kube.<namespace_name>.<container_name>.<pod_name>.<docker_id>-
        Tag_Regex         (?<pod_name>[a-z0-9](?:[-a-z0-9]*[a-z0-9])?(?:\\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?<namespace_name>[^_]+)_(?<container_name>.+)-(?<docker_id>[a-z0-9]{64})\.log$
        Path              /var/log/containers/*.log
        Parser            docker
        Refresh_Interval  10 

  filter.conf: |
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_Tag_Prefix     kube.
        Regex_Parser        kube-tag

  output.conf: |
    [OUTPUT]
        Name            stdout        
        Format          json_lines
        Match           kube.kiamol-ch13-test.*

  parsers.conf: |
    [PARSER]
        Name        docker
        Format      json
        Time_Key    time
        Time_Format %Y-%m-%dT%H:%M:%S.%L
        Time_Keep   On
    
    [PARSER]
        Name    kube-tag
        Format  regex
        Regex   ^(?<namespace_name>[^_]+)\.(?<container_name>.+)\.(?<pod_name>[a-z0-9](?:[-a-z0-9]*[a-z0-9])?(?:\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)\.(?<docker_id>[a-z0-9]{64})-$
```

### Elastic Search에 로그 저장

* 다음 Deployment 정의를 통해 Elastic Search 파드를 구동한 후, fluentd의 ConfigMap을 수정해 Elastic Search에 로그를 출력하도록 한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  namespace: kiamol-ch13-logging
  labels:
    kiamol: ch13
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - image: kiamol/ch13-elasticsearch
        name: elasticsearch
        ports:
        - containerPort: 9200
          name: elasticsearch
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  namespace: kiamol-ch13-logging
  labels:
    kiamol: ch13
spec:
  selector:    
    app: elasticsearch
  ports:
  - name: elasticsearch
    port: 9200
    targetPort: 9200
  type: ClusterIP
```

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: kiamol-ch13-logging
  labels:
    kiamol: ch13
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     error
        Daemon        off
        Parsers_File  parsers.conf

    @INCLUDE input.conf
    @INCLUDE filter.conf
    @INCLUDE output.conf

  input.conf: |
    [INPUT]
        Name              tail
        Tag               kube.&#x3C;namespace_name>.&#x3C;container_name>.&#x3C;pod_name>.&#x3C;docker_id>-
        Tag_Regex         (?&#x3C;pod_name>[a-z0-9](?:[-a-z0-9]*[a-z0-9])?(?:\\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?&#x3C;namespace_name>[^_]+)_(?&#x3C;container_name>.+)-(?&#x3C;docker_id>[a-z0-9]{64})\.log$
        Path              /var/log/containers/*.log
        Parser            docker
        Refresh_Interval  10 

  filter.conf: |
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_Tag_Prefix     kube.
        Regex_Parser        kube-tag

<strong>  output.conf: |
</strong><strong>    [OUTPUT] # 테스트 네임스페이스의 로그는 elastic search의 test 인덱스에 저장
</strong><strong>        Name            es
</strong><strong>        Match           kube.kiamol-ch13-test.*
</strong><strong>        Host            elasticsearch
</strong><strong>        Index           test
</strong><strong>        Generate_ID     On
</strong>
<strong>    [OUTPUT] # 시스템 파드의 로그는 elastic search의 sys 인덱스에 저장
</strong><strong>        Name            es
</strong><strong>        Match           kube.kube-system.*
</strong><strong>        Host            elasticsearch
</strong><strong>        Index           sys
</strong><strong>        Generate_ID     On
</strong>
  parsers.conf: |
    [PARSER]
        Name        docker
        Format      json
        Time_Key    time
        Time_Format %Y-%m-%dT%H:%M:%S.%L
        Time_Keep   On
    
    [PARSER]
        Name    kube-tag
        Format  regex
        Regex   ^(?&#x3C;namespace_name>[^_]+)\.(?&#x3C;container_name>.+)\.(?&#x3C;pod_name>[a-z0-9](?:[-a-z0-9]*[a-z0-9])?(?:\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)\.(?&#x3C;docker_id>[a-z0-9]{64})-$
</code></pre>

* ConfigMap을 변경하더라도 파드는 재시작되지 않으므로, 직접 재시작해주어야 한다.

```
kubectl apply -f fluent-config-elasticsearch.yaml

kubectl rollout restart ds/fluent-bit -n kiamol-ch13-logging
```

* fluentd와 elastic search를 사용하면 로그 데이터 처리 파이프라인을 애플리케이션과 분리하여 처리할 수 있기 때문에 좋다.
* 예를 들어 특정 로그 레벨만 로깅하도록 변경하고 싶을 때, 애플리케이션 자체에서 로그 출력을 조절할 수도 있지만 애플리케이션 재시작이 필요하다. fluentd에서 로그를 필터링해 중앙 저장소에 저장한다면, 애플리케이션 재시작 없이 로그를 필터링할 수 있다.
* 다음은 필터링 설정의 예시이다. 이를 통해 priority 필드값이 2, 3, 4인 경우에만 로그를 중앙 저장소에 저장하게 된다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: kiamol-ch13-logging
  labels:
    kiamol: ch13
data:
  fluent-bit.conf: |
  # ...
  filter.conf: |
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_Tag_Prefix     kube.
        Regex_Parser        kube-tag
        Annotations         Off
        Merge_Log           On
        K8S-Logging.Parser  On

<strong>    [FILTER]
</strong><strong>        Name                grep
</strong><strong>        Match               kube.kiamol-ch13-test.api.numbers-api*
</strong><strong>        Regex               priority [234]
</strong></code></pre>

### 다양한 로그 모델

* fluentd는 다양한 기능을 제공하고 효율적으로 동작하지만 복잡한 로그 처리 파이프라인에는 연산 시간이 소요되어 입출력 사이에 지연이 발생할 수 있다.
* 애플리케이션에서 중앙 로그 저장소로 곧바로 로그를 기록하거나 사이드카 컨테이너를 이용해 중앙 로그 저장소에 기록하도록 직접 구현할 수 있다.

## 모니터링

* 중앙화된 시스템에서 측정값을 수집하여 전체 애플리케이션 컴포넌트의 상태를 파악할 수 있다.
* 모니터링 시 측정되어야 할 주요 정보로는 latency, traffic, error, saturation들이 있다. 하지만 가장 중요한 것은 사용자 경험 관점에서의 애플리케이션 성능에 영향을 미치는 원인에 지표를 찾는 것이다.
* 쿠버네티스에서는 프로메테우스와 연동해 클러스터의 측정값을 수집하고 저장할 수 있다.
* 프로메테우스에서는 서비스의 로드밸런싱을 통해 메트릭 정보를 얻는 것보다 각 파드의 IP 주소를 알아내 메트릭 정보를 알아내어야 한다. 이러한 과정 속에서 사용자에게 노출하는 API와 쿠버네티스 내부에서만 사용하는 API를 제어할 수도 있을 것이다.
* 프로메테우스 deployment에서 사용되는 ConfigMap에는 스크래핑 대상을 정의해야 한다.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kiamol-ch14-monitoring
data:
  prometheus.yml: |-
    global:
      scrape_interval: 30s
     
    scrape_configs:
      - job_name: 'test-pods'
        kubernetes_sd_configs:
        - role: pod 
        relabel_configs:
        - source_labels: 
            - __meta_kubernetes_namespace
          action: keep
          regex: kiamol-ch14-test 
```

* 여기서 스크래핑 대상이 된 애플리케이션은 스스로 메트릭 값을 구해 이 정보를 HTTP로 가져갈 수 있도록 해야 한다.
* 메트릭을 제공하는 주소가 `/metrics` 와 다르다면 직접 HTTP 주소를 아래와 같이 어노테이션을 통해 정의해주어야 한다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: apod-api
  namespace: kiamol-ch14-test
spec:
  selector:
    matchLabels:
      app: apod-api
  template:
    metadata:
      labels:
        app: apod-api
      annotations:
<strong>        prometheus.io/path: "/actuator/prometheus"
</strong>    spec:
      containers:
        - name: api
          image: kiamol/ch14-image-of-the-day
          ports:
            - containerPort: 80
              name: api
</code></pre>

* 이후 프로메테우스 정의에서 어노테이션에 해당하는 API를 사용할 수 있도록 아래와 같이 정의한다.
  * 프로메테우스의 모든 규칙은 레이블 형태로 처리되어 `meta_kubernetes_pod_annotation_<어노테이션 이름>` 레이블로 변환된다.
  * `meta_kubernetes_pod_annotationpresent_<어노테이션 이름>` 레이블을 통해서 해당 어노테이션이 존재하는 지 확인할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kiamol-ch14-monitoring
data:
  prometheus.yml: |-
    global:
      scrape_interval: 30s
     
    scrape_configs:
      - job_name: 'test-pods'
        kubernetes_sd_configs:
        - role: pod 
        relabel_configs:
<strong>        - source_labels: 
</strong><strong>            - __meta_kubernetes_pod_annotationpresent_prometheus_io_path
</strong><strong>            - __meta_kubernetes_pod_annotation_prometheus_io_path
</strong><strong>          regex: true;(.*)
</strong><strong>          target_label:  __metrics_path__
</strong></code></pre>

*   만약 프로메테우스 스크래핑 대상에서 제외하려면 아래와 같이 파드 정의를 하면 된다.

    <pre class="language-yaml"><code class="lang-yaml">  template:
        metadata:
          labels:
            app: proxy
          annotations:
    <strong>        prometheus.io/scrape: "false"
    </strong></code></pre>
* 프로메테우스의 지표들을 시각화하기 위해서는 그라파나를 파드로 띄워야 한다.&#x20;
* 모니터링을 위해서는 1. 시스템 상태를 파악하는 데에 필요한 정보 목록을 작성하고, 2. 개발 및 운영 팀에서 해당 정보를 추출하는 모니터링 시스템을 구축해야 한다.

### 사이드카 컨테이너로 측정값을 추출하기

* 프로메테우스가 인식할 수 있는 형태의 측정값을 제공하지 못하는 애플리케이션들은 프로메테우스가 인식할 수 있는 형태의 측정값을 제공하는 사이드카 컨테이너와 함께 사용될 수 있다.
* Nginx용 프로메테우스 지표 추출기를 사용하면, 같은 파드 내에 존재하는 Nginx 서버에 localhost로 접근해 지표를 수집하고 HTTP 엔드포인트로 추출한 측정값을 내보낸다.
* 아래는 Nginx용 프로메테우스 지표 추출을 위한 사이드카 컨테이너 정의이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-proxy
  namespace: kiamol-ch14-test
spec:
  selector:
    matchLabels:
      app: todo-proxy
  template:
    metadata:
      labels:
        app: todo-proxy
<strong>      annotations:
</strong><strong>        prometheus.io/port: "9113"
</strong>    spec:
      containers:
        - name: nginx
          image: nginx:1.17-alpine          
          ports:
            - name: http
              containerPort: 80              
          volumeMounts:
            - name: config
              mountPath: "/etc/nginx/"
              readOnly: true
<strong>        - name: exporter
</strong><strong>          image: nginx/nginx-prometheus-exporter:0.8.0
</strong><strong>          ports:
</strong><strong>            - name: metrics
</strong><strong>              containerPort: 9113
</strong><strong>          args:
</strong><strong>            - -nginx.scrape-uri=http://localhost/stub_status
</strong>      volumes:
        - name: config
          configMap:
            name: todo-proxy-config
              
</code></pre>

* 애플리케이션 자체에서 측정값을 제공하지 않으며 쿠버네티스의 자기수복 기능을 사용할 수 없는 경우, 프로메테우스에서 제공하는 [블랙박스 exporter](https://github.com/prometheus/blackbox_exporter)를 사이드카 컨테이너로 등록해 TCP/HTTP 요청으로 애플리케이션의 정상 여부를 확인할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: numbers-api
  namespace: kiamol-ch14-test
spec:
  selector:
    matchLabels:
      app: numbers-api
  template:
    metadata:
      labels:
        app: numbers-api
<strong>      annotations:
</strong><strong>        prometheus.io/port: "9115"
</strong><strong>        prometheus.io/path: "/probe"
</strong><strong>        prometheus.io/target: "http://127.0.0.1/healthz"
</strong>    spec:
      containers:
        - name: api
          image: kiamol/ch03-numbers-api
          ports:
            - containerPort: 80
              name: api
          env:
            - name: FailAfterCallCount
              value: "3"
<strong>        - name: exporter
</strong><strong>          image: prom/blackbox-exporter:v0.17.0
</strong><strong>          ports:
</strong><strong>            - name: metrics
</strong><strong>              containerPort: 9115
</strong></code></pre>

### 쿠버네티스 객체와 컨테이너 모니터링

* 쿠버네티스 객체와 컨테이너 상태 정보는 쿠버네티스 API를 통해 직접 수집할 수 없다.
* 데몬셋 형태로 각 노드에 배치되어 노드의 컨테이너 런타임에서 정보를 수집하는 **cAdvisor**와 쿠버네티스 API에서 정보를 수집하는 **kube-state-metrics** 도구를 이용해야 한다.
* 다음은 데몬셋 타입의 cAdvisor의 정의이다.

```yaml
apiVersion: apps/v1 
kind: DaemonSet
metadata:
  name: cadvisor
  namespace:  kube-system
  labels:
    kiamol: ch14
spec:
  selector:
    matchLabels:
      app: cadvisor
  template:
    metadata:
      labels:
        app: cadvisor
    spec:
      containers:
      - name: cadvisor
        image: k8s.gcr.io/cadvisor:v0.35.0
        volumeMounts:
        - name: rootfs
          mountPath: /rootfs
          readOnly: true
        - name: var-run
          mountPath: /var/run
          readOnly: true
        - name: sys
          mountPath: /sys
          readOnly: true
        - name: docker
          mountPath: /var/lib/docker
          readOnly: true
        - name: disk
          mountPath: /dev/disk
          readOnly: true
        ports:
          - name: http
            containerPort: 8080
            protocol: TCP
      automountServiceAccountToken: false
      volumes:
      - name: rootfs
        hostPath:
          path: /
      - name: var-run
        hostPath:
          path: /var/run
      - name: sys
        hostPath:
          path: /sys
      - name: docker
        hostPath:
          path: /var/lib/docker
      - name: disk
        hostPath:
          path: /dev/disk
```

* 다음은 디플로이먼트 타입의 kube-state-metrics 정의이다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kube-state-metrics
  namespace: kube-system
  labels:
    kiamol: ch14
spec:
  ports:
  - name: http-metrics
    port: 8080
    targetPort: http-metrics
  - name: telemetry
    port: 8081
    targetPort: telemetry
  selector:
    app: kube-state-metrics
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: kube-system
  labels:
    kiamol: ch14
spec:
  selector:
    matchLabels:
      app: kube-state-metrics
  template:
    metadata:
      labels:
        app: kube-state-metrics
    spec:
      serviceAccountName: kube-state-metrics
      containers:
      - image: quay.io/coreos/kube-state-metrics:v1.9.7
        name: kube-state-metrics
        ports:
        - containerPort: 8080
          name: http-metrics
        - containerPort: 8081
          name: telemetry
---
# RBAC configuration - ignore this until we get to chapter 17 :)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-state-metrics
  namespace: kube-system
  labels:
    kiamol: ch14
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-state-metrics
  labels:
    kiamol: ch14
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - secrets
  - nodes
  - pods
  - services
  - resourcequotas
  - replicationcontrollers
  - limitranges
  - persistentvolumeclaims
  - persistentvolumes
  - namespaces
  - endpoints
  verbs:
  - list
  - watch
# ...
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-state-metrics
  labels:
    kiamol: ch14
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
subjects:
- kind: ServiceAccount
  name: kube-state-metrics
  namespace: kube-system
```

* cAdvisor, kube-state-metrics의 지표들을 스크래핑하기 위한 prometheus ConfigMap 정의이다. ConfigMap이 변경되더라도 파드가 자동으로 반영하지 않으므로 `curl -X POST $(kubectl get svc prometheus -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:9090/-/reload' -n kiamol-ch14-monitoring)` 명령을 이용해 갱신된 설정값을 사용하도록 해야 한다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kiamol-ch14-monitoring
data:
  prometheus.yml: |-
    global:
      scrape_interval: 30s
     
    scrape_configs:      
<strong>      - job_name: 'cadvisor'
</strong>        kubernetes_sd_configs:
        - role: pod 
        relabel_configs:
        - source_labels: 
            - __meta_kubernetes_namespace
            - __meta_kubernetes_pod_labelpresent_app
            - __meta_kubernetes_pod_label_app
          action: keep
          regex: kube-system;true;cadvisor
<strong>      - job_name: 'kube-state-metrics'
</strong>        static_configs:
        - targets:
            - kube-state-metrics.kube-system.svc.cluster.local:8080
            - kube-state-metrics.kube-system.svc.cluster.local:8081
</code></pre>
