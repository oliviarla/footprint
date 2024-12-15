# Helm

## 헬름의 기능

* 여러 개의 YAML 정의 스크립트를 하나의 아티팩트로 묶어 관리할 수 있다.
* 헬름 명령을 통해 복잡한 애플리케이션을 구성하는 쿠버네티스 리소스들을 쉽게 배치할 수 있다.
* 애플리케이션 수준의 추상화를 추가해준다.
* 헬름 레포지토리는 도커 허브 같은 컨테이너 이미지 레지스트리와 비슷한데, 서버에서 사용 가능한 모든 패키지의 레이블을 제공하여 이를 로컬에 저장할 수 있다.

```
# 헬름 레포지토리를 추가
helm repo add <로컬 이름> <원격 레포지토리>

# 레포지토리를 업데이트하여 인덱스를 캐싱
helm repo update

# 인덱스 캐시로부터 애플리케이션 검색
helm search repo <애플리케이션 이름> --versions
```

* 헬름에서 **차트**란 애플리케이션의 패키지를 의미한다. 차트를 설치하면 릴리스라고 부르며, 하나의 클러스터에 같은 차트를 여러 개 설치할 수 있다.
* 애플리케이션의 매니페스트 파일들과 파라미터값, 템플릿 변수 등을 디렉터리 혹은 압축 파일로 묶어 차트를 만들 수 있다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* 차트에 포함된 파라미터 값의 기본값을 확인할 수 있으며, 설치 시 기본값을 수정할 수 있다.

```
helm show values <레포지토리 이름>/<헬름 차트 이름> --version <버전>

helm install --set servicePort=8080 --set replicaCount=1 <헬름 프로세스 이름> <레포지토리 이름>/<헬름 차트 이름> --version <버전>
```

* 설정값 파일을 이용해 차트를 설치할 수도 있다.

```
helm install -f <설정값 파일> <헬름 프로세스 이름> <레포지토리 이름>/<헬름 차트 이름>
```

* 차트가 설치되면 내부적으로 정의한 쿠버네티스 리소스들을 kubectl 명령을 이용해 확인할 수 있다.

```
kubectl get deploy -l app.kubernetes.io/instance=<헬름 프로세스 이름> --show-labels

kubectl get rs -l app.kubernetes.io/instance=<헬름 프로세스 이름>
```

* 다음 명령을 이용해 헬름 차트 프로세스를 업데이트할 수 있다.

```
helm upgrade --set servicePort=8080 --set replicaCount=3 <헬름 프로세스 이름> <레포지토리 이름>/<헬름 차트 이름> --version <버전>
```

## 애플리케이션 패키징

* 템플릿 변수를 이용해 여러 쿠버네티스 객체에서 설정값을 가져올 수 있다.
* 템플릿 변수는 두 겹 중괄호로 표현되며, 아래의 경우 Release, Values 객체에서 설정값을 참조한다. Release 객체는 install/upgrade 명령 실행 시 관련 정보를 담아 생성되는 객체이고, Values 객체는 차트에 포함된 기본값에 사용자가 지정한 값을 오버라이드하여 생성되는 객체이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    kiamol: {{ .Values.kiamolChapter }}
# ...
```

* `helm create` 명령을 이용해 새로운 차트 생성 시 템플릿을 생성할 수 있다.
* `helm lint` 명령을 이용해 차트에 들어갈 파일의 유효성을 검증할 수 있다.
* `helm ls` 명령을 이용해 현재 설치된 차트 목록을 확인할 수 있다.
* 헬름에서는 따옴표를 앞뒤로 붙여주거나 반복, 분기 로직 등의 다양한 템플릿 함수를 제공하여 템플릿 값을 가공할 수 있다.
* 정의에서 유일한 값을 갖는 부분을 템플릿 변수로 만들면 헬름을 이용해 동일한 애플리케이션을 여러 개 수행시킬 수 있다.
* 헬름 차트에 포함된 템플릿을 대상으로 kubectl apply 명령을 사용할 수 없다. 템플릿 변수로 인해 유효한 YAML 파일이 되지 못하기 때문이다. 즉, 쿠버네티스 정의와 헬름 정의는 호환되지 못한다.

### 차트 배포

* 레포지토리 전용 서버인 차트뮤지엄을 설치해 사용하면 로컬에서 간단히 리포지토리를 등록할 수 있다.
* 차트의 배포는 보통 아래 3단계로 이뤄진다.
  * 차트를 zip 파일로 압축한다.
  * 서버에 압축 파일을 업로드한다.
  * 레포지토리 인덱스에 새로운 차트 정보를 추가한다. (이 부분은 차트 뮤지엄 등 레포지토리 서버가 대신 처리할 것이다.)

## 차트 간 의존 관계 모델링

* 차트 간 의존 관계는 유연해야 한다.
* 상위 차트가 하위 차트를 필요로 해야 하며, 하위 차트는 독립적으로 사용 가능해야 한다.
* 하위 차트는 일반적으로 활용될 수 있도록 템플릿 변수를 사용해야 한다.
* 상위 차트의 정의에서 하위 차트를 의존 차트 목록에 추가해야 한다. 그리고 하위 차트의 설정값을 상위 차트 정의에서 지정해야 한다.
* 다음은 proxy, vweb 차트에 의존하는 pi 차트 정의와 pi 차트의 값 정의 예제이다.

```yaml
apiVersion: v2
name: pi
description: A Pi calculator
type: application
version: 0.1.0
dependencies: 
  - name: vweb
    version: 2.0.0
    repository: https://kiamol.net
    condition: vweb.enabled
  - name: proxy
    version: 0.1.0
    repository: file://../proxy
    condition: proxy.enabled
```

```yaml
# number of app Pods to run
replicaCount: 2
# type of the app Service:
serviceType: LoadBalancer
# settings for vweb
vweb:
   # whether to deploy vweb
   enabled: false    
# settings for the reverse proxy
proxy:
  # whether to deploy the proxy
  enabled: false
  # name of the app Service to proxy
  upstreamToProxy: "{{ .Release.Name }}-web"
  # port of the proxy Service
  servicePort: 8030
  # number of proxy Pods to run
  replicaCount: 2
```

* 상위 차트는 지정된 버전의 하위 차트와 함께 패키징된다. 버전 수정 없이 차트를 업데이트하면 의존 관계에서 최신 상태를 받아볼 수 없게 되므로 항상 업데이트 시 버전 수정이 필요하다.

## 헬름으로 설치한 릴리즈의 업그레이드와 롤백

* 쿠버네티스 정의에 따라 전략이 결정된다.
* 클러스터에 애플리케이션을 한 세트 더 설치해 문제가 없는지 테스트해볼 수 있다.
* `--atomic` 플래그를 제공하여 helm upgrade 시 원자적으로 롤백해주는 기능을 제공한다. 따라서 모든 리소스의 업데이트가 끝나기를 기다렸다가 만약 실패한 리소스가 있다면 다른 리소스들을 이전 상태로 되돌린다.
* `helm history <헬름 프로세스 이름>` 명령을 통해 설치 히스토리를 자세히 확인할 수 있다.
* `helm rollback <헬름 프로세스 이름> --revision <리비전 번호>` 명령을 이용해 특정 리비전으로 롤백할 수 있다.
