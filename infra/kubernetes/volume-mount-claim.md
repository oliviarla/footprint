# Volume, Mount, Claim

## 쿠버네티스에서의 컨테이너 파일 시스템 구축 과정

* 컨테이너에서 동작하는 애플리케이션에는 읽기 쓰기가 가능한 하나의 파일 시스템으로 보이지만, 실제로 파드 속 컨테이너의 파일 시스템은 여러 출처를 합쳐 구성된다.
* 컨테이너 이미지가 파일 시스템 초기 내용을 제공하고, 컨테이너가 기록 가능한 **writable layer**가 얹혀진다. 컨테이너 이미지에 들어 있던 파일을 수정하는 것은 writable layer에서 원본 파일의 사본을 수정하는 것이다.
* 기존 컨테이너에서 동작하던 애플리케이션에서 장애가 발생해 컨테이너가 종료되면 새로운 파드가 생성되고 기존 컨테이너의 writable layer에 기록한 데이터가 유실된다.
* 컨피그맵과 비밀값을 통해 파드 수준에서 볼륨을 정의하고 컨테이너 파일 시스템 내부로 마운트할 수 있다. 컨피그맵과 비밀값은 읽기 전용 스토리지 단위이며, 이외에도 기록 가능한 유형의 볼륨이 여러 가지 있다.

### EmptyDir

* 컨테이너 안에서 빈 디렉터리로 초기화되는 유형의 볼륨을 사용할 수 있다.
* 파드 수준의 스토리지이며, 컨테이너에 마운트되므로 디렉터리처럼 보이지만 이미지나 컨테이너 레이어에 속하지 않는다.
* 임시 저장 목적으로 모든 애플리케이션에서 사용할 수 있다. 상당 시간동안 유효한 API 응답이 있다면 이를 로컬 캐싱해두어 애플리케이션에 장애가 생겨 파드가 재시작되거나 애플리케이션이 느릴 때 응답을 빠르게 줄 수 있다.
* EmptyDir 볼륨에 저장된 데이터는 컨테이너가 재시작되어도 유지된다. 파드가 대체될 때에는 데이터가 유실된다.
* 아래는 디플로이먼트에 EmptyDir 볼륨을 정의하는 예제이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
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
              mountPath: /data
      volumes:
        - name: data
          emptyDir: {}
```

* 아래와 같은 명령을 사용하면 EmptyDir 볼륨을 포함한 디플로이먼트를 적용한 후 emptyDir 볼륨이 마운트된 위치에 파일을 생성하고 컨테이너를 강제 종료해 파드가 재구동되어도 파일이 남아있는지 확인할 수 있다.

```bash
kubectl apply -f sleep-with-emptydir.yaml

kubectl exec deploy/sleep -- ls /data

kubectl exec deploy/sleep --sh -c 'echo ch05 > /data/file.txt; ls /data'

# 컨테이너 ID 확인
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'

# 컨테이느 프로세스 강제 종료
kubectl exec deploy/sleep --killall5

# 변경된 컨테이너 ID 확인
kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'

kubectl exec deploy/sleep -- cat /data/file.txt
```

## 볼륨과 마운트로 노드에 데이터 저장

* 데이터를 특정 노드에 고정시킬지 말지 결정해야 한다. 만약 특정 노드에 고정시키면 기존 파드를 대체할 새로운 파드가 동일 노드에만 배치되야 한다.
* 노드의 디스크를 가리키는 볼륨인 호스트 경로 볼륨은 파드에 정의되며 컨테이너 파일 시스템에 마운트되는 형태로 사용된다.
* 쿠버네티스가 클러스터의 모든 노드에 동일한 데이터를 복제해주지는 않는다.
* 파드가 항상 같은 노드에서 동작하는 한 볼륨의 생애 주기가 노드의 디스크와 같아진다.
* 아래는 원주율 계산 결과를 반환하는 웹 애플리케이션의 프록시 파드를 정의한 디플로이먼트로, 컨테이너에서 `/data/nginx/cache` 디렉토리에 데이터를 저장하면 노드의 `/volumes/nginx/cache` 디렉토리에 실제로 데이터가 기록되도록 정의했다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-proxy
  labels:
    app: pi-proxy
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
              mountPath: /data/nginx/cache # 프록시의 캐시 저장 경로
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath:
            path: /volumes/nginx/cache # 사용할 노드의 디렉토리
            type: DirectoryOrCreate # 디렉터리가 없으면 생성
```

* 위 디플로이먼트를 적용하면 컨테이너가 종료될 때 대체 파드가 기존 파드와 같은 노드에 뜨기만 하면 같은 볼륨을 그대로 사용할 수 있다.
* 하지만 대부분의 경우 노드를 여러 개 두고 운영하고, 항상 같은 노드에서 수행하도록 강제하면 노드가 고장났을 때 애플리케이션을 실행할 수 없으므로 안정성이 떨어지게 된다.
* 노드 파일 시스템 전체에 접근할 수 있는 파드를 띄울 수 있다면 누구든지 호스트경로 볼륨에 사용하는 노드 상의 디렉터리에 접근할 수 있게 된다. 애플리케이션이 침투당해 공격자가 컨테이너에서 명령을 실행할 수 있다면 노드의 디스크 전체를 장악당할 수 있다.
* 아래는 모든 노드 파일 시스템에 접근 가능하도록 만든 디플로이먼트 예시와, 명령을 통해 내부 데이터를 조회하는 명령 예시이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
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
            - name: node-root
              mountPath: /node-root
      volumes:
        - name: node-root
          hostPath:
            path: /
            type: Directory
```

```bash
# 컨테이너 속 로그 파일 확인
kubectl exec deploy/sleep -- ls -l /var/log

# 노드 파일 시스템의 로그 파일 확인
kubectl exec deploy/sleep -- ls -l /node-root/var/log
```

* 볼륨 마운트 시 볼륨의 하위 디렉토리를 마운트해 노드의 파일 시스템을 필요 이상으로 노출하지 않아야 한다.
* 아래는 볼륨 마운트를 하위 디렉터리만 대상으로 하여 컨테이너에서 접근 가능한 노드의 디렉토리를 제한하는 예시이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
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
            - name: node-root # 마운트할 볼륨 이름
              mountPath: /pod-logs # 마운트 대상 컨테이너 경로
              subPath: var/log/pods # 마운트 대상 볼륨 내 경로
            - name: node-root
              mountPath: /container-logs
              subPath: var/log/containers
      volumes:
        - name: node-root
          hostPath:
            path: /
            type: Directory
```

## 영구 볼륨과 클레임

* 분산 스토리지 시스템에 모든 노드가 접근 가능하면 파드가 어떤 노드에서 실행되더라도 볼륨에 접근할 수 있다.
* 파드는 컴퓨팅 계층의 추상이며, 서비스는 네트워크 계층의 추상이다. 스토리지 계층의 추상으로는 영구 볼륨(PersistentVolume, PV)과 영구 볼륨 클레임(PersistentVolumeClaim, PVC)이 있다.
* 영구 볼륨이란 사용 가능한 스토리지 조각을 정의한 쿠버네티스 리소스이다. 클러스터 관리자가 생성하며 각 영구 볼륨에는 이를 구현하는 스토리지 시스템에 대한 볼륨 정의가 들어 있다.
* 아래는 NFS 스토리지를 사용하는 영구 볼륨 정의의 예시이다.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01 # 볼륨 이름
spec:
  capacity:
    storage: 50Mi # 볼륨 용량
  accessModes:
    - ReadWriteOnce # 파드 접근 유형
  nfs:
    server: nfs.my.network # NFS 서버의 도메인 이름
    path: "/kubernetes-volumes" # 스토리지 경로
```

* 모든 노드에서 접근 가능한 NFS 혹은 애저 디스크를 이용하면 PV 정의에서 분산 스토리지 유형을 지정하기만 하면 되지만, 노드가 하나인 로컬에서는 로컬 볼륨을 정의하고 레이블로 볼륨 노드를 따로 식별해야 한다.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01
spec:
  capacity:
    storage: 50Mi
  accessModes:
    - ReadWriteOnce
  local:
    path: /volumes/pv01 
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
          - key: kiamol
            operator: In
            values:
              - ch05
```

```bash
# 클러스터의 첫 노드에 레이블 부여
kubectl label node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') kiamol=ch05

# 레이블 셀렉터로 노드 조회
kubectl get nodes -l kiamol=ch05

# 레이블이 부여된 노드의 로컬 볼륨 사용하도록 영구 볼륨 배치
kubectl apply -f persistentVolume.yaml
```

* **파드는 PV에 직접 접근하지 못하며 PVC에 볼륨 사용을 요청해야 한다.**
* PVC는 파드가 사용하는 스토리지 추상이며, 애플리케이션에서 사용할 스토리지를 요청한다. PVC는 요구 조건이 일치하는 영구 볼륨과 함께 쓰이며, 자세한 볼륨 정보는 PV에 위임한다.
* PV와 PVC는 일대일 관계이며 하나의 PVC에 연결된 PV는 다른 PVC와 연결될 수 없다.
* 아래는 앞서 생성한 로컬 볼륨과 요구 조건이 일치하는 PVC 정의이다. 스토리지 유형을 지정하지 않으면 현재 존재하는 PV 중 요구사항과 일치하는 것을 자동으로 찾아준다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce # 접근 유형
  resources:
    requests:
      storage: 40Mi # 스토리지 용량
  storageClassName: "" # 스토리지 유형
```

```bash
# PVC 생성
kubectl apply -f persistentVolumeClaim.yaml

# PVC 목록 확인 / STATUS가 BOUND이면 PV에 연결되어 사용중임을 의미
kubectl get pvc

# PV 목록 확인 / STATUS가 BOUND이면 PVC에 연결되어 사용중임을 의미
kubectl get pv
```

* PVC를 볼륨으로 정의하려면 아래와 같이 디플로이먼트에 볼륨을 정의해주어야 한다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-db
spec:
  selector:
    matchLabels:
      app: todo-db
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
            - name: data
              mountPath: /var/lib/postgresql/data
      volumes:
        # ...
<strong>        - name: data
</strong><strong>          persistentVolumeClaim: # PVC를 볼륨으로 사용한다.
</strong><strong>            claimName: postgres-pvc # PVC 이름
</strong></code></pre>

* 노드에 볼륨이 사용할 디렉토리 경로를 만들어주어야 한다. 이 때 노드에 로그인할 권한이 없으므로 hostPath를 통해 노드의 루트 디렉토리를 마운트한 파드를 이용해 디렉토리를 생성할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
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
            - name: node-root
              mountPath: /node-root
      volumes:
        - name: node-root
          hostPath:
            path: /
            type: Directory
```

```
kubectl apply -f sleep-with-hostpath.yaml

kubectl exec deploy/sleep -- mkdir -p /node-root/volumes/pv01
```

* 전체적인 구성은 아래 그림과 같다.

<figure><img src="../../.gitbook/assets/image (3).png" alt="" width="336"><figcaption></figcaption></figure>

## 스토리지 유형과 동적 볼륨 프로비저닝

* 앞서 살펴본 방식은 PV를 명시적으로 생성하는 정적 프로비저닝 방식이었다. 요구 사항이 일치하는 PV가 없으면 PVC가 생성되기는 하지만 스토리지를 사용할 수 없고 대기 상태가 된다.
* 정적 볼륨 프로비저닝 방식은 모든 쿠버네티스 클러스터에서 사용할 수 있으며 스토리지 접근 제약이 큰 조직에서 선호하는 방식이다.
* 동적 볼륨 프로비저닝 방식은 PVC만 생성하면 그에 맞는 PV를 클러스터에서 동적으로 생성해준다.
* 스토리지 유형을 지정하거나 따로 지정하지 않고 기본 유형으로 사용할 수 있다.
* PV를 따로 만들지 않아도 PVC를 배치할 수 있다.
* 다음은 스토리지 유형을 지정하지 않은 PVC의 정의이다. PVC를 생성하면 클러스터에 지정된 기본 스토리지 유형의 PV가 자동으로 생성될 것이다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  # storageClassName: hostpath
  resources:
    requests:
      storage: 100Mi
```

* 아래 명령을 통해 PVC를 생성하고 제거할 때 PV가 동적으로 생성되고 제거됨을 확인할 수 있다.

```bash
# PVC 생성
kubectl apply -f persistentVolumeClaim-dynamic.yaml

# PVC, PV 목록 확인
kubectl get pvc
kubectl get pv

# PVC 제거
kubectl delete pvc persistentVolumeClaim-dynamic.yaml

# PV도 같이 제거되었는지 확인
kubectl get pv
```

* 스토리지 유형은 표준 쿠버네티스 리소스로 생성되며 다음 세 가지 필드로 스토리지의 동작 방식을 지정할 수 있다.
  * provisioner
    * 영구 볼륨이 필요할 때 영구 볼륨을 만드는 주체. 플랫폼에 따라 관리 주체가 달라진다.
  * reclaimPolicy
    * 연결되었던 클레임이 삭제되었을 때 남아있는 볼륨을 어떻게 처리할지 정한다.
  * volumeBindingMode
    * PVC가 생성되자마자 PV를 생성해 연결할지, PVC를 사용하는 파드가 생성될 때 PV를 생성할 지 정한다.

## 스토리지 선택 시 고려할 점

* 일반적인 경우 파드 정의에 포함된 PVC에서 필요한 스토리지 용량과 스토리지 유형을 지정하면 된다.
* 클라우드 환경 사용 시 스토리지 솔루션은 비용이 많이 든다. 속도가 빠른 스토리지 유형을 지정한 PVC에서 기존 볼륨을 유지하도록 설정하면, PVC를 사용하는 파드가 삭제되더라도 볼륨이 유지되므로 비용이 계속 청구될 것이다.
* 데이터베이스 같은 유상태 애플리케이션의 경우 쿠버네티스에서 데이터 복제 기능과 함께 고가용성을 확보할 수 있지만, 데이터의 백업과 스냅샷 남기기와 복원 등 직접 고려해야할 점이 많다.
