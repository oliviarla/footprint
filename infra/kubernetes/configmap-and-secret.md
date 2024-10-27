# ConfigMap & Secret

## 애플리케이션에 설정이 전달되는 과정

* 쿠버네티스에서 컨테이너에 설정값을 주입할 때 사용할 수 있는 리소스에는 컨피그맵(ConfigMap)과 비밀값(Secret)이 있다.
* 클러스터의 다른 리소스와 독립적인 공간에 보관된다.
* 파드 정의에서 컨피그맵과 비밀 값의 데이터를 읽어오도록 할 수 있다.
* 다른 리소스들과 달리 스스로 어떤 기능을 갖지 않으며 데이터를 저장하는 것만이 목적이다.
* 아래와 같이 파드 정의에 환경 변수를 추가하여 간단히 설정값을 주입할 수 있다.

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
          env: # 환경변수 정의
          - name: KIAMOL_CHAPTER
            value: "04"
```

### 컨피그맵

* 컨피그맵은 파드에서 읽어들일 데이터를 저장하는 리소스이다.
* key-value, 텍스트, 바이너리 파일 등의 데이터 형태가 될 수 있다.
* 컨피그맵은 특정 파드 전용으로 사용할 수도 있고 여러 파드에서 공유할 수도 있다. 단, 파드에서 컨피그맵 내용을 수정할 수는 없다.
* 아래와 같이 컨피그맵을 생성할 수 있다.

```
kubectl create configmap sleep-config-literal --from-literal=kiamol.section='4.1'
```

* 아래와 같이 컨피그맵에 들어 있는 데이터나 상세 정보를 확인할 수 있다.

```
kubectl get cm sleep-config-literal
kubectl describe cm sleep-config-literal
```

* 아래와 같이 파드의 정의에서 컨피그맵 이름과 읽어들일 항목 이름을 지정할 수 있다.

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
          env:
          - name: KIAMOL_SECTION
            valueFrom:
              configMapKeyRef:              
                name: sleep-config-literal
                key: kiamol.section
```

## 컨피그맵에 저장한 설정 파일 사용하기

* env 파일에 아래와 같이 환경 변수가 저장되어 있다면 해당 내용을 컨피그맵으로 만들 수 있다.

```
KIAMOL_CHAPTER=ch04
KIAMOL_SECTION=ch04-4.1
KIAMOL_EXERCISE=try it now
```

```
kubectl create configmap sleep-config-env-file --from-env-file=sleep/ch04.env
```

* 우선순위가 다르게 부여된 출처 별로 설정값을 읽어들일 수 있다. 쿠버네티스에서는 애플리케이션에 다음 전략을 사용한다.
  * 기본 설정은 이미지에 포함시킨다.
  * 각 환경 별 설정값은 컨피그맵에 담겨서 컨테이너의 파일 시스템으로 전달된다. 애플리케이션에서 지정한 경로에 설정파일을 주입하거나, 컨테이너 이미지에 담긴 파일을 덮어쓰는 방식이다.
  * 변경이 필요한 설정값은 디플로이먼트 내 파드 정의에서 환경변수 형태로 적용한다.
* 아래와 같이 컨피그맵을 정의하면 컨테이너 이미지에 포함된 JSON 설정 파일에 설정값을 추가 적용되도록 할 수 있다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: todo-web-config-dev
data:
  config.json: |-
    {
      "ConfigController": {
        "Enabled" : true
      }
    }
```

* 컨피그맵을 apply한 후에는 컨피그맵을 참조하는 pod도 다시 apply해주어야 한다.

```
# 컨피그맵 apply
kubectl apply -f todo-list/configMaps/todo-web-config-dev.yaml
# 컨피그맵을 참조하는 deployment apply
kubectl apply -f todo-list/todo-web-dev.yaml
```

## 컨피그맵에 담긴 설정값 데이터 주입하기

* 환경 변수 대신 **컨테이너 파일 시스템 속 파일로 설정값을 주입**할 수 있다.
* 컨테이너 파일 시스템 구성에 컨피그맵을 추가할 수 있다.
* **컨피그맵은 디렉토리 형태로, 내부의 항목들에 대해서는 파일 형태로 컨테이너 파일 시스템에 추가**된다.
* 볼륨은 컨피그맵에 담긴 데이터를 파드로 전달한다.
* 볼륨 마운트는 컨피그맵을 읽어들인 볼륨을 파드 컨테이너의 특정 경로에 위치시킨 읽기 전용 데이터이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-web
spec:
  selector:
    matchLabels:
      app: todo-web
  template:
    metadata:
      labels:
        app: todo-web
    spec:
      containers:
        - name: web
          image: kiamol/ch04-todo-list 
          volumeMounts:                  # 컨테이너에 볼륨을 마운트
            - name: config               # 마운트할 볼륨 이름
              mountPath: "/app/config"   # 볼륨 마운트 경로
              readOnly: true    
      volumes:                           # 볼륨은 파드 수준에서 정의한다.
        - name: config                   
          configMap:                     # 볼륨의 원본은 컨피그맵이다.
            name: todo-web-config-dev
```

* 컨테이너를 디렉토리 형태로 읽어들이면 다양한 애플리케이션 설정을 적용할 수 있다. 즉, 여러 설정들을 하나의 컨피그맵으로 관리할 수 있다.





## 비밀값을 사용해 민감한 정보 다루기



## 애플리케이션 설정 관리





