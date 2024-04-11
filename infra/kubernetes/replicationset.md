# ReplicationSet

## ReplicaSet

* 파드를 특정 개수만큼 복제하여 항상 유지하도록 한다.

### 설정 파일

* `apiVersion` 은 apps/v1으로 설정해야 한다.
* metadata에는 ReplicaSet의 정보를 담는다.
* spec에는 template, replicas, selector라는 세가지 설정을 해주어야 한다.
  * template에는 복제할 대상 파드를 정의해야 한다.
  * replicas에는 복제할 개수를 지정해야 한다.
  * selector에는 클러스터에 존재하는 여러 파드 중 어떤 레이블을 가진 파드만 모니터링할 지 지정해야 한다.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: myapp
    tier: frontend
spec:
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
```

### 명령어

* 설정 파일을 사용해 ReplicaSet을 생성 및 실행시킬 수 있고, 모든 ReplicaSet 정보를 얻을 수 있으며, 제거도 가능하다.

```
kubectl create -f replicaset.yml

kubectl get replicaset

kubectl delete replicaset <replicaset name>
```

### Scale

* ReplicaSet의 복제본 개수를 늘리거나 줄일 수 있다.
* 아래는 설정 파일을 수정하고 다시 적용하는 명령어이다.

```
kubectl replace -f replicaset.yml
```

* 혹은 설정 파일 변경 없이 복제본의 개수를 변경할 수 있다. 다만 설정 파일에 자동으로 이 정보가 업데이트되지는 않는다.

```
kubectl scale --replicas=6 -f replicaset.yml
```

### Replication Controller와 비교

* Replication Controller는 ReplicaSet 이전에 존재하던 방식으로, 현재는 ReplicaSet으로 대체되었다.
* 사용 방식은 거의 유사하다.

```yaml
apiVersion: v1 # apps/v1이 아니다
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
