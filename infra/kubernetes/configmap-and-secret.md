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
* 설정값을 컨피그맵에 일괄 저장해두고 애플리케이션 정의를 분리하면 각 팀이 각자의 담당 부분을 처리할 수 있어 배포 시 유연성이 생긴다.
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

#### 파일로 설정값 주입하기

* 환경 변수 대신 **컨테이너 파일 시스템 속 파일로 설정값을 주입**할 수 있다.
* 쿠버네티스는 컨테이너 파일 시스템 구성에 컨피그맵을 추가할 수 있다.
* **컨피그맵은 디렉토리 형태로, 내부의 항목들에 대해서는 파일 형태로 컨테이너 파일 시스템에 추가**된다.
* 볼륨은 컨피그맵에 담긴 데이터를 파드로 전달한다.
* 볼륨 마운트는 컨피그맵을 읽어들인 볼륨을 파드 컨테이너의 특정 경로에 위치시킨 읽기 전용 데이터이다. 아래 예시와 같이 볼륨 마운트 경로를 설정하면 해당 경로에 컨피그맵 디렉토리가 생기고 각 설정 파일들이 저장된다.

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

* 컨테이너를 디렉토리 형태로 읽어들이는 이유는 여러 설정들을 하나의 컨피그맵으로 관리하기 위함이다.
* 파드가 동작중일 때 컨피그맵을 업데이트하면 쿠버네티스가 수정된 파일을 컨테이너에 전달한다. 이 후의 과정은 애플리케이션이 파일을 처리하는 방식에 달려있다. 애플리케이션이 변경 감지하도록 설계했다면 애플리케이션에 반영될 것이고, 최초에만 설정을 사용한다면 별다른 일이 발생하지 않을 것이다.
* 아래는 컨피그맵을 적용한 후 컨테이너 내부 파일에 잘 저장되었는지 확인하는 명령어들이다.

```bash
# 변경된 configMap 적용
kubectl apply -f todo-list/configmaps/todo-web-config.yaml

# configMap이 파드에 반영될 때 까지 잠깐 대기
sleep 120

# configMap으로부터 파일이 생성되었는지 확인하기
kubectl exec deploy/todo-web --sh -c 'ls -l /app/config'
```

> **mountPath는** 기존 데이터를 제거하고 컨피그맵 볼륨을 저장하기 때문에 **이미 데이터가 들어있는 디렉토리를 지정하면 안된다**. 애플리케이션 바이너리 파일이 들어있을 경우 파드가 재실행될 수 없어 오류와 함께 종료된다.

#### 필요한 데이터 항목만 대상 디렉터리에 전달하기

* volumes 내부에 items를 통해 원하는 컨피그맵의 데이터 항목 하나를 원하는 파일에 저장할 수 있도록 지정할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
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
          volumeMounts:
            - name: config
              mountPath: "/app/config"
              readOnly: true
      volumes:
        - name: config
          configMap:
            name: todo-web-config-dev
<strong>            items:
</strong><strong>              - key: config.json # 데이터 항목 이름
</strong><strong>                path: config.json # 저장할 파일 이
</strong></code></pre>

## 비밀값을 사용해 민감한 정보 다루기

* 비밀값은 해당 값을 사용해야 하는 노드에만 전달되며, 디스크에 저장하지 않고 메모리에만 담긴다.
* 전달 과정과 저장할 때 모두 암호화가 적용된다.
* 아래와 같은 명령을 이용해 비밀값을 생성하고 확인할 수 있다.

```
# 시크릿 생성
kubectl create secret generic <비밀값 이름> --from-literal=<비밀값 항목 key>=<비밀값 항목 value>

# 시크릿 상세 정보 확인
kubectl describe secret <비밀값 이름>

# 시크릿 내용의 평문 확인을 위해 base64 디코딩 수행
kubectl get secret <비밀값 이름> -o jsonpath='{.data.secret}' | base64 -d
```

### 비밀값을 환경 변수로 전달하기

* 파드 정의에서는 아래와 같이 secretKeyRef를 이용해 비밀값을 환경 변수로 주입할 수 있다.

```
spec:
  containers:
    - name: sleep
      image: kiamol/ch03-sleep
      env:
      - name: KIAMOL_SECRET
        valueFrom:
        secretKeyRef:
          name: <비밀값 이름>
          key: <비밀값 항목 key>
```

* 아래 명령을 이용해 디플로이먼트를 재적용한 후 파드의 환경 변수에 접근해 비밀값이 잘 적용되었는지 확인할 수 있다.

```
# 디플로이먼트 적용
kubectl apply -f sleep/sleep-with-secret.yaml

# 파드 속 환경 변수 확인 (비밀값 데이터의 평문이 출력된다)
kubectl exec deploy/sleep -- printenv KIAMOL_SECRET
```

* 환경 변수는 컨테이너에서 동작하는 모든 프로세스에서 접근 가능하고, 간혹 애플리케이션에서 치명적 오류가 발생했을 때 모든 환경 변수를 로그로 남기는 경우가 있다.

### 비밀값을 파일로 전달하기

* 비밀값을 환경 변수 대신 파일 형태로 전달하고 파일 권한을 볼륨에서 설정하면 더욱 안전하다.
* 다음은 비밀값을 정의하는 예제이다.
  * 비밀값을 YAML로 관리하면 일관적인 애플리케이션 배치가 가능하지만 github 같은 형상 관리 도구에 노출되므로 github secret 등으로 추가적인 정보 보호를 해주어야 한다.

```yaml
apiVersion: v1
kind: Secret # 비밀값 유형으로 리소스 생성
metadata:
  name: todo-db-secret-test
type: Opaque # 임의의 텍스트 데이터를 담고자 Opaque 타입 선택
stringData: # 텍스트 데이터
  POSTGRES_PASSWORD: "kiamol-2*2*" # 저장할 데이터 (key-value)
```

* 아래 명령들을 통해 비밀값을 생성하고 인코딩된 데이터 값을 확인할 수 있다.

```bash
# 비밀값 생성
kubectl apply -f todo-db-secret-test.yaml

# 인코딩된 데이터 값 확인
kubectl get secret todo-db-secret-test -o jsonpath='{.data.POSTGRES_PASSWORD}'
```

* 비밀값의 데이터로 json 타입도 넣을 수 있다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: todo-web-secret-test
type: Opaque
stringData:
  secrets.json: |-
    {
      "ConnectionStrings": {
        "ToDoDb": "Server=todo-db;Database=todo;User Id=postgres;Password=kiamol-2*2*;"
      }
    }
```

* 아래는 정의된 비밀값을 볼륨으로 마운트하는 파드 정의의 예시이다. 이 파드를 배치하면 /secrets/postgres\_password 파일에 비밀값 데이터가 전달되고 0400 권한이므로 컨테이너 사용자만 읽을 수 있다. db 파드는 비밀값 파일을 환경 변수로 가져와 내부적으로 사용할 것이다.

```yaml
spec:
  containers:
    - name: db
      image: postgres:11.6-alpine
      env:
      - name: POSTGRES_PASSWORD_FILE        # 설정파일이 마운트될 경로
        value: /secrets/postgres_password
      volumeMounts:
        - name: secret                      # 마운트할 볼륨 이름
          mountPath: "/secrets"             # 마운트 되는 경로
  volumes:
    - name: secret
      secret:                               # 비밀값에서 볼륨 생성
        secretName: todo-db-secret-test     # 볼륨을 만들 비밀값 이름
        defaultMode: 0400                   # 파일 권한 설정
        items:                              # 비밀값의 특정 데이터 항목 지정
        - key: POSTGRES_PASSWORD
          path: postgres_password
```

## 애플리케이션 설정 관리

### 설정 업데이트

* 애플리케이션 중단 없이 설정 변경에 대응이 필요하다면, 기존 컨피그맵이나 시크릿을 업데이트하는 방식과 함께 볼륨 마운트를 이용해 설정 파일을 수정해야 한다.
* 애플리케이션을 업데이트할 때 새로운 설정 객체를 배치한 후 이를 가리키도록 애플리케이션 정의를 수정할 수도 있다. 이 과정에서 파드 교체가 발생하지만, 설정값 변경의 이력이 남고 만일의 사태에 이전 설정으로 돌아갈 수 있다는 장점이 있다.

### 민감 정보 관리

* 형상 관리 도구에 저장된 YAML 템플릿 파일로 컨피그맵과 비밀 값 정의를 두고, 외부에 절대 노출되지 말아야 하는 정보는 Azure KeyVault 등에 보관해두고 필요할 때 YAML 파일에 채워 사용할 수 있다.
* 설정 파일 배포를 관리하는 설정 관리 전담 팀이 있다면 컨피그맵과 비밀 값의 버전 관리 정책을 사용하면 된다. 외부에 절대 노출되지 말아야 하는 정보는 관리 팀에서 별도 시스템을 통해 저장해두고 필요할 때 kubectl create로 설정 객체를 생성해 사용할 수 있다.
* 아래는 위 두 방식을 그림으로 표현한 것이다.

<figure><img src="../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>
