# Configuration

* 쿠버네티스 객체(pod, replicas, deployment services 등)를 생성하려면 YAML 파일을 만들어야 한다.
* YAML 파일의 템플릿은 아래와 같다.
  * `apiVersion` 에서는 사용할 쿠버네티스 API의 버전을 명시한다.
  * `kind` 에서는 생성할 객체의 타입을 지정한다.
  * `metadata` 에서는 이름, 레이블 등의 정보를 지정한다. 레이블의 경우 사용자가 커스텀한 key-value 데이터를 설정해 사용할 수 있다. 예를 들어 frontend, backend, db와 같은 타입을 지정하기 위해 type이라는 키를 두고 value에 타입 정보를 담아 분류에 사용되도록 할 수 있다.
  * `spec` 에서는 컨테이너에 대한 정보를 지정한다. 컨테이너는 여러 개를 지정할 수 있다.
* 아래는 YAML 파일의 예제이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
    - name: nginx-container
      image: nginx
```

* 아래 명령을 통해 YAML 파일을 통해 쿠버네티스 객체를 생성해 실행할 수 있다.

```
kubectl apply -f pod-definition.yml
```

* 간단한 설정만 필요한 경우 자동으로 yaml 파일을 만들고 실행할 수도 있다.

```
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
```
