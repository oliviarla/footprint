# Rollout & Rollback

## 롤아웃과 롤백으로 디플로이먼트 업데이트

* 상황에 따라 업데이트를 중지하거나 롤백이 가능한 롤링 업데이트를 이용하면, 업데이트로 인한 애플리케이션 중단 시간을 감소시키고 안정적인 운영이 가능해진다.
* 디플로이먼트 정의를 수정해 반영할 때 마다 쿠버네티스는 롤아웃을 수행한다. 롤아웃은 레플리카셋을 새로 만들어 지정된 수 만큼 레플리카 수를 늘리고, 기존 레플리카 셋의 레플리카 수를 0으로 만드는 방식으로 진행된다.
* 롤아웃은 파드의 정의가 변경될 때에만 발생한다.
* `kubectl apply` 명령을 실행시킬 때 --record 플래그를 이용해 해당 명령이 기록되도록 할 수 있다. 이를 통해 `kubectl rollout history` 명령에서 롤아웃이 실행된 kubectl 명령을 확인할 수 있다.

```
kubectl apply -f vweb-v11.yaml --record

kubectl rollout history deploy/<디플로이먼트 이름>
```

* 다음 명령을 통해 롤아웃 상태를 확인할 수 있다.

```
kubectl rollout status deploy/<디플로이먼트 이름> --timeout=2s
```

* 이전 리비전으로 돌리거나 특정 리비전으로 롤백할 수 있다.
  * \--dry-run 옵션을 사용하면 롤백을 실제로 수행하지 않고, 예상 결과를 확인할 수 있다.
  * \--to-revision 옵션을 사용해 특정 리비전으로 롤백할 수 있다.

```
kubectl rollout undo deploy/<디플로이먼트 이름>

kubectl rollout undo deploy/<디플로이먼트 이름> --dry-run

kubectl rollout undo deploy/<디플로이먼트 이름> --to-revision=2
```

* 컨피그맵과 비밀값을 연동한 디폴로이먼트에서 컨피그맵을 업데이트하더라도 디플로이먼트에는 변경이 없으므로 **이전 컨피그맵에 대한 롤백이 불가능**하다. 하지만 설정값만 업데이트할 때에는 롤아웃이 발생하지 않으므로 서비스 중단의 위험이 없고 매끄러운 업데이트가 가능하다.

```
kubectl apply -f deploy-with-configmap.yaml --record

kubctl apply -f update-configmap.yaml --record
```

{% code title="deploy-with-configmap.yaml" %}
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vweb-config
  labels:
    kiamol: ch09
data:
  v.txt:
    v3-from-config
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vweb
  labels:
    kiamol: ch09
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vweb
  template:
    metadata:
      labels:
        app: vweb
        version: v3
    spec:
      containers:
        - name: web
          image: kiamol/ch09-vweb:v2
          ports:
            - name: http
              containerPort: 80
          volumeMounts:
            - name: static
              mountPath: "/usr/share/nginx/html/"
              readOnly: true
      volumes:
        - name: static
          configMap:
            name: vweb-config
```
{% endcode %}

{% code title="update-configmap.yaml" %}
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vweb-config
  labels:
    kiamol: ch09
data:
  v.txt:
    v3.1
```
{% endcode %}

* 컨피그맵과 비밀값을 불변으로 간주하여 버전 명명 규칙을 따라 이름을 짓고 설정값이 변경되면 새로운 객체를 만들어 디플로이먼트의 참조를 변경하도록 할 수 있다. 이를 통해 컨피그맵을 변경할 때 마다 디플로이먼트가 업데이트되므로 롤아웃 히스토리가 남아 롤백이 가능하다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: ConfigMap
metadata:
<strong>  name: vweb-config-v4
</strong>  labels:
    kiamol: ch09
data:
  v.txt:
    v4-from-config
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vweb
  labels:
    kiamol: ch09
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vweb
  template:
    metadata:
      labels:
        app: vweb
        version: v4
    spec:
      containers:
        - name: web
          image: kiamol/ch09-vweb:v2
          ports:
            - name: http
              containerPort: 80
          volumeMounts:
            - name: static
              mountPath: "/usr/share/nginx/html/"
              readOnly: true
      volumes:
        - name: static
<strong>          configMap:
</strong><strong>            name: vweb-config-v4
</strong></code></pre>

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: ConfigMap
metadata:
<strong>  name: vweb-config-v41
</strong>  labels:
    kiamol: ch09
data:
  v.txt:
    v4.1-from-config
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vweb
  labels:
    kiamol: ch09
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vweb
  template:
    metadata:
      labels:
        app: vweb
        version: v4.1
    spec:
      containers:
        - name: web
          image: kiamol/ch09-vweb:v2
          ports:
            - name: http
              containerPort: 80
          volumeMounts:
            - name: static
              mountPath: "/usr/share/nginx/html/"
              readOnly: true
      volumes:
        - name: static
<strong>          configMap:
</strong><strong>            name: vweb-config-v41
</strong></code></pre>

## 디플로이먼트 롤링 업데이트 설정

* 디플로이먼트 업데이트 전략은 롤링 업데이트와 리크리에이트가 있다.

### 롤링 업데이트 전략

* 롤링 업데이트는 기존 레플리카셋의 레플리카 수를 점차 줄이는 동시에 새로운 레플리카셋의 레플리카 수를 늘린다.
* maxUnavailable 값으로 업데이트 동안 **사용할 수 없는 파드의 최대 수**를 지정할 수 있다. 즉, 기존 레플리카 셋에서 동시에 종료되는 파드의 수를 지정해 스케일링 속도를 조절할 수 있다.
* maxSurge 값으로 업데이트 동안 생기는 **잉여 파드의 최대 수**를 지정할 수 있다. 즉, 새 레플리카 셋에서 동시에 새로 시작되는 파드의 수를 지정해 스케일링 속도를 조절할 수 있다.
* 예를 들어 replicas가 3이고 maxSurge는 1, maxUnavailable은 0으로 설정하면 디플로이먼트는 최대 네 개의 레플리카를 동시에 가질 수 있다. 따라서 새로운 레플리카 셋의 파드가 하나 먼저 만들어진 후 기존 레플리카 셋의 파드를 제거하게 된다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* maxSurge가 1, maxUnavailable이 1이면 기존 파드가 하나 삭제되는 동시에 새로운 파드가 2개 생성된다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 다음 두 값을 통해 롤아웃 속도를 조절할 수 있다.
  * minReadySeconds로 지정된 값의 시간동안 오류로 종료되는 컨테이너가 없어야 파드가 안정적이라고 판정한다. 이를 통해 신규 파드 상태가 안정적인지 확인할 수 있는 시간 여유를 둘 수 있다.
  * progressDeadlineSeconds로 지정된 값 안에 신규 파드 상태가 안정되지 않는다면 해당 파드가 실패했다고 간주한다.
* 파드가 실패하면 새 파드를 만들어 계속 재시도하는데, 재시도 사이에 일정 시간 간격을 점차 지수적으로 늘려가는 백오프 시간을 둔다.

### 리크리에이트 전략

* 리크리에이트는 기존 레플리카셋의 레플리카가 완전히 없어진 후 새로운 레플리카셋의 레플리카 수를 늘린다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vweb
  labels:
    kiamol: ch09
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vweb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: vweb
        version: v2
    spec:
      containers:
        - name: web
          image: kiamol/ch09-vweb:v2
          ports:
            - name: http
              containerPort: 80
```

* 리크리에이트 전략을 사용할 경우 업데이트 배치 전 테스트로 충분히 검증되어야 한다. 만약 그렇지 않다면 새 파드에서 오류가 발생했을 때 기존 파드들이 사라진 상태이므로 애플리케이션 서비스가 중단될 수 있기 때문이다.

## 데몬셋과 스테이트풀셋 롤링 업데이트

### onDelete

* 각 파드의 업데이트 시점을 직접 제어해야 할 때 사용하는 전략이다.
* 업데이트 시 컨트롤러가 기존 파드를 종료하지 않고 그대로 둔 상태에서 다른 프로세스가 파드를 삭제하면 새로운 파드를 생성한다.
* 예를 들어 제거되기 전 가지는 모든 데이터를 디스크에 기록해야하는 파드를 스테이트풀셋이 관리하고 있다면, 모든 작업을 마치고 파드를 직접 제거하여 업데이트 시점을 제어할 수 있다.

### 데몬셋 업데이트

* 데몬셋은 클러스터의 모든 노드에 파드를 하나씩만 실행하므로, 업데이트 시 잉여 파드를 만들 수 없어 파드를 삭제 후 새로 띄우는 전략만 사용 가능하다. 즉, maxSurge 값을 지정할 수 없다.
* 이로 인해 노드가 하나라면 대체 파드가 생성될 때 까지 애플리케이션을 사용할 수 없는 상태가 된다.
* 노드가 여러 개라면 파드가 한 노드에서 하나씩 업데이트되므로 항상 최소 하나의 파드가 준비 상태를 유지한다.
* maxUnavailable 값을 조정해 동시에 업데이트할 파드 개수를 조절할 수 있으나 여러 파드를 한 번에 제거하면 대체 파드가 생성될 때 까지 처리량이 감소한다.

### 스테이트풀셋 업데이트

* maxSurge, maxUnavailable 값을 사용할 수 없으며 동시에 업데이트되는 파드 수는 항상 하나이다.
* partition값을 이용해 전체 파드 중 업데이트해야 하는 파드의 비율을 정할 수 있다. 예를 들어 5개의 레플리카가 있고 partition을 3으로 지정하면 3, 4번째 파드만 업데이트된다.
* partition 값을 차츰 줄여가며 릴리즈의 페이스를 조절하다가 안정적임을 확인하면 0으로 설정해 전체 스테이트풀셋을 업데이트할 수 있다. 예를 들어 데이터베이스의 복제 노드를 먼저 업데이트하고 웹 애플리케이션을 읽기 전용으로 잠시 변경한 후, 데이터베이스의 마스터 노드를 업데이트하고 다시 웹 애플리케이션을 정상화할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: todo-db
  labels:
    kiamol: ch09
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo-db
  serviceName: todo-db  
<strong>  updateStrategy:
</strong><strong>    type: RollingUpdate
</strong><strong>    rollingUpdate:
</strong><strong>      partition: 1  # only updates Pod 1
</strong>      # ...
</code></pre>

## 릴리즈 전략

### 블루-그린 배치 전략

* 애플리케이션의 두 버전을 각각 별도 디플로이먼트 객체로 실행시키고 쿠버네티스 서비스의 레이블 셀렉터를 이용해 구버전과 신버전을 전환할 수 있다.
* 모든 파드가 준비된 상태에서 트래픽을 전달받으므로 한 순간에 업데이트가 일어나며, 레이블 셀렉터만 변경하면 롤백할 수 있다.
* 컴퓨팅 파워를 많이 소모하고 롤아웃 히스토리가 남지 않는다.
