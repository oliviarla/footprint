# Controller

## 애플리케이션 스케일링 방법

* 쿠버네티스는 컴퓨팅 계층에서 네트워크와 스토리지 계층을 추상화시켰기 때문에 동일한 애플리케이션이 돌아가는 여러 파드(레플리카, Replica)를 구동할 수 있다.
* 노드가 여러 개인 클러스터에서 레플리카는 여러 노드에 분산 배치된다.
* 파드를 관리하는 리소스를 **컨트롤러**라고 한다. 따라서 컨트롤러 리소스 정의에는 파드의 템플릿이 포함된다.
* 컨트롤러 리소스는 파드를 생성하고 대체하는 데에 템플릿을 사용한다.
* 레플리카셋(ReplicaSet)은 파드를 직접 관리하는 역할을 한다.
  * 레플리카 수가 변경되면 파드를 추가/감소시킨다.
  * 기존 파드가 사라지면 대체 파드를 생성한다.
  * 관리하던 컨테이너가 종료되면 대체 컨테이너를 생성한다.
* 아래 그림처럼 디플로이먼트는 레플리카셋을 관리하고, 레플리카셋은 파드를 관리한다.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* 레플리카셋의 YAML 정의 예시이다. replicas 필드를 통해 파드 수를 지정할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
<strong>kind: ReplicaSet
</strong>metadata:
  name: whoami-web
  labels:
    kiamol: ch06
spec:
<strong>  replicas: 1
</strong>  selector:
    matchLabels:
      app: whoami-web
  template:
    metadata:
      labels:
        app: whoami-web
    spec:
      containers:
        - image: kiamol/ch02-whoami
          name: web
          ports:
            - containerPort: 80
              name: http
</code></pre>

* 다음 명령들을 이용해 레플리카셋을 배치시키고 상태를 확인할 수 있다.

```bash
# 레플리카셋 배치
kubectl apply -f whoami.yaml

# 레플리카셋 리소스 확인 / 자신이 관리하는 파드의 이상적인 상태와 현재 상태 확인 가능
kubectl get replicaset whoami-web

# 레플리카셋 정보 확인
kubectl describe rs whoami-web
```

* 레플리카셋은 항상 제어 루프를 돌며 관리중인 리소스와 필요한 리소스 수를 확인하므로 파드가 삭제되는 즉시 대체 파드를 구동시킨다.

```bash
# 기존 파드 목록 확인
kubectl get pods -l app=whoami-web

# 기존 파드 모두 제거
kubectl delete pods -l app=whoami-weeb

# 새로운 파드 목록 확인
kubectl get pods -l app=whoami-web
```

* 노드가 여러 개인 운영 환경에서는 노드에서 컨테이너 이미지를 처음 내려받는다면 시간이 조금 걸리므로 스케일링이 적용되는데 걸리는 시간에 영향을 미친다. 따라서 이미지 최적화가 필요하다.
* 레플리카셋과 서비스를 함께 사용하면 레이블 셀렉터를 통해 HTTP 요청이 들어올 때마다 여러 레플리카에 요청을 고르게 분배되도록 할 수 있다.
* 아래는 서비스 정의 예시이다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami-web
  labels:
    kiamol: ch06
spec:
  ports:
    - port: 8088
      targetPort: 80
  selector:
    app: whoami-web
  type: LoadBalancer
```

* 레플리카셋과 클러스터IP를 함께 사용하여도 마찬가지로 로드밸런싱이 적용된다.

## 디플로이먼트와 레플리카셋

#### 디플로이먼트

* 디플로이먼트는 레플리카셋 위에 유용한 관리 계층을 추가한다. 컨트롤러 리소스로는 디플로이먼트를 먼저 고려해야 한다.
* 디플로이먼트는 여러 레플리카셋을 관리할 수 있다. 레플리카셋 버전업을 하고자 한다면, 기존 레플리카셋의 파드 수를 0으로 줄이고 새로운 레플리카셋의 파드 수를 늘리면 된다.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

* 디플로이먼트에 스케일링을 적용하려면 레플리카셋과 마찬가지로 replicas 필드를 둘 수 있다. 생략 시 기본값 1이 사용된다.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-web
  labels:
    kiamol: ch06
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pi-web
  template:
    metadata:
      labels:
        app: pi-web
    spec:
      containers:
        - image: kiamol/ch05-pi
          command: ["dotnet", "Pi.Web.dll", "-m", "web"]
          name: web
          ports:
            - containerPort: 80
              name: http
```

* 디플로이먼트의 레이블 셀렉터는 파드 템플릿의 레이블과 일치해야 한다.
* 디플로이먼트에서 파드 정의를 변경하면, 새로운 대체 레플리카셋을 생성하고 기존 레플리카 셋의 replicas를 0으로 만든다.
* 이를 통해 디플로이먼트는 애플리케이션 업데이트 과정 혹은 컨테이너에 발생하는 문제 처리 과정을 유연하게 대처할 수 있다.

#### 컨트롤러 리소스에 스케일링 바로 지시

* kubectl scale 명령을 통해 만약의 경우 스케일링할 수 있다.
* YAML 파일을 사용하여 형상 관리 도구와 애플리케이션 배치 상태를 동일하게 유지하는 것이 좋다.
* 만약 애플리케이션 성능 문제가 발생했고 자동 재배치가 오래 걸리는 상황이라면 즉시 kubectl scale 명령으로 대응한 후 YAML 파일을 함께 수정할 수 있다.

```
# 파드 개수를 4개로 늘린다
kubectl scale --replicas=4 deploy/pi-web

# 파드 개수가 4개임을 확인
kubectl get rs -l app=pi-web

# 기존 yaml의 로그 수준만 업데이트한 후 다시 반영
kubectl apply -f pi-web.yaml

# 파드 개수가 기존대로 3개임을 확인
kubectl get rs -l app=pi-web
```

#### 여러 파드 간 볼륨 공유하기

* 아래와 같이 hostpath 볼륨을 사용하도록 디플로이먼트를 정의하면, 같은 노드에서 실행된 모든 파드들이 같은 볼륨을 사용하도록 할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-proxy
  labels:
    kiamol: ch06
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pi-proxy
  template:
    metadata:
      labels:
        app: pi-proxy
    spec:
      containers:
        - image: nginx:1.17-alpine
          name: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: "/etc/nginx/"
              readOnly: true
            - name: cache-volume
              mountPath: /data/nginx/cache
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
<strong>          hostPath:
</strong><strong>            path: /volumes/nginx-cache
</strong><strong>            type: DirectoryOrCreate
</strong></code></pre>

## 데몬셋으로 고가용성 확보하기

### 데몬셋

* 데몬셋이란 클러스터 내 모든 노드 또는 셀렉터와 일치하는 일부 노드에서 단일 레플리카 또는 파드로 동작하는 리소스를 의미한다.
* 각 노드에서 정보를 수집해 중앙의 수집 모듈에 전달하거나 인프라 수준의 관심사(파드의 로그나 노드 활성 지표 등)와 관련된 목적으로 많이 쓰인다.
* 한 노드에 하나의 레플리카만 두면 되는 경우 데몬셋을 활용할 수 있다. 리버스 프록시를 제공하는 nginx 같은 경우 파드 하나로도 수천 개의 요청을 동시에 처리할 수 있다.
* hostpath 볼륨을 사용할 경우 각 파드가 자신만의 캐시를 갖게 된다. 상태를 가진 애플리케이션의 경우 같은 데이터 파일에 동시에 접근할 인스턴스가 하나 뿐이므로 데몬셋으로 구동시키기 좋다.
* 아래와 같이 데몬셋을 정의할 수 있다. 노드가 새로 추가될 때마다 파드가 추가된 노드에 구동될 것이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
<strong>kind: DaemonSet
</strong>metadata:
  name: pi-proxy
  labels:
    kiamol: ch06
spec:
  selector:
    matchLabels:
      app: pi-proxy # 레이블을 기준으로 데몬셋의 관리 대상 파드를 결정한다
  template:
    metadata:
      labels:
        app: pi-proxy
    spec:
      containers:
        - image: nginx:1.17-alpine
          name: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: "/etc/nginx/"
              readOnly: true
            - name: cache-volume
              mountPath: /data/nginx/cache
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath:
            path: /volumes/nginx-cache
            type: DirectoryOrCreate
</code></pre>

* 서비스를 통해 외부 요청을 받고 있는 디플로이먼트를 데몬셋으로 변경하려면, 디플로이먼트를 제거하기 전 데몬셋을 먼저 생성하여 서비스의 엔드포인트에 추가되도록 한다. 이후 디플로이먼트를 삭제하면 된다.

```bash
# 데몬셋 배치
kubectl apply -f nginx-ds.yaml

# 서비스의 엔드포인트에 추가되었는지 확인
kubectl get endpoints pi-proxy

# 디플로이먼트 삭제
kubectl delete deploy pi-proxy

# 데몬셋 상태 확인
kubectl get daemonset pi-proxy

# 디플로이먼트의 파드가 삭제되는지 확인
kubectl get po -l app=pi-proxy
```

### 데몬셋과 노드셀렉터 함께 사용하기

* 원하는 노드에만 파드를 실행할 수 있도록 노드셀렉터를 통해 노드를 선택할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: pi-proxy
  labels:
    kiamol: ch06
spec:
  selector:
    matchLabels:
      app: pi-proxy
  template:
    metadata:
      labels:
        app: pi-proxy
    spec:
      containers:
        - image: nginx:1.17-alpine
          name: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: "/etc/nginx/"
              readOnly: true
            - name: cache-volume
              mountPath: /data/nginx/cache
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath:
            path: /volumes/nginx-cache
            type: DirectoryOrCreate
<strong>      nodeSelector:
</strong><strong>        kiamol: ch06
</strong></code></pre>

## 쿠버네티스 객체 간 오너십
