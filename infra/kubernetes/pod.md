# Pod

## 컨테이너 실행 및 관리 방법

* 쿠버네티스는 직접 컨테이너를 실행하는 대신 컨테이너를 생성할 책임을 해당 노드에 설치된 컨테이너 런타임에 위임한다.
* 파드는 쿠버네티스가 관리하는 리소스이고, 컨테이너는 쿠버네티스의 외부에서 관리된다.
* 파드는 원시 타입 리소스이므로, 일반적으로는 직접 실행하는 대신 컨트롤러 객체를 만들어 파드를 관리하도록 한다.
* 노드는 파드에 포함된 컨테이너를 실행하는 책임을 갖는다. 컨테이너 런타임 인터페이스를 통해 어떤 컨테이너 런타임과도 연동될 수 있다.
* 쿠버네티스는 컨테이너가 다운되어 파드 상태가 변화하면 새로운 컨테이너를 추가해 파드 상태를 복원하는 형태로 자기수복성을 제공한다.
* 쿠버네티스는 컨테이너 생성 시 파드 이름을 컨테이너 레이블에 추가한다. 이를 통해 도커에서 파드에 포함된 컨테이너를 찾을 수 있다.

```
docker container ls -q --filter label=io.kubernetes.container.name=<podname>
```

## 파드

### 개념

* 쿠버네티스는 워커 노드에 바로 컨테이너를 실행시키지 않는다. 대신 파드(Pod)라는 독립된 애플리케이션 위에서 컨테이너를 동작시킨다. 따라서 모든 컨테이너는 파드 라는 단위에 속한다.
* 파드는 쿠버네티스에서 생성 가능한 가장 작은 단위이다.

### 특징

* 하나의 파드는 대부분 하나의 컨테이너와 매핑되지만, 옵션을 설정하여 여러 컨테이너를 실행시킬 수도 있다.
* 자신만의 가상 IP 주소를 가지며, 이 주소를 통해 가상 네트워크에 접속된 다른 파드와 통신할 수 있다.
* 하나의 파드에 포함된 컨테이너들은 같은 가상 환경에 포함되므로 localhost로 통신 가능하다.
* 파드는 생성 시에 하나의 노드에 배정이 된다. 노드는 파드를 관리하고 파드에 포함된 컨테이너를 실행하는 책임을 갖는다.
* 스케일아웃을 위해 새로운 컨테이너를 실행하고자 하는 경우에는 새로운 파드를 생성해 컨테이너를 실행해야 한다.
* 하지만 사용자가 업로드한 파일 데이터를 프로세싱하는 등 애플리케이션을 지원하는 작업을 하는 컨테이너를 실행시킬 때에는 애플리케이션 컨테이너와 동일한 파드 내에서 동작하도록 할 수 있다.

### 명령어

* 도커 이미지를 통해 Pod를 실행시킬 수 있다.

```
kubectl run <podname> --image=<docker image name>
```

* 현재 실행중인 Pod의 목록을 확인할 수 있다. -o wide 옵션을 통해 ip, 노드 정보도 확인할 수 있다.

```
kubectl get pods
kubectl get pods -o wide
```

* 현재 실행중인 특정 파드의 정보를 확인할 수 있다.

```
kubectl get pod <podname>
```

* 실행중인 pod의 노드 정보, 컨테이너 정보, 발생한 Events 등 상세 정보를 확인할 수 있다.

```
kubectl describe pod <podname>
```

* 네트워크 트래픽을 노드에서 파드로 전달하도록 포트포워딩하여 파드에 요청을 보낼 수 있다.

```
kubectl port-forward pod/<podname> <localhost port>:<pod port>
```

* 모든 파드를 삭제할 수 있다.

```
kubectl delete pods --all
```

## 디플로이먼트

* 쿠버네티스의 컨트롤러는 다른 리소스를 관리하는 쿠버네티스 리소스이다.
* 현재 상태를 감시하다가 사용자가 정의한 상태와 차이가 생기면 차이를 없애도록 한다.
* 디플로이먼트는 **파드를 주로 관리하는 컨트롤러 객체**이다.
* 어떤 노드에 장애가 발생해 파드가 유실되면 대체 파드를 다른 노드에 실행한다.
* 필요한 파드 수를 지정해 여러 노드에 걸쳐 파드가 실행되도록 할 수 있다.
* 아래와 같이 디플로이먼트를 생성할 수 있다. 파드의 복제본 수를 지정하지 않으면 기본적으로 한 개 생성된다.

```
kubectl create deployment <deployment name> --image=<docker image>
```

* 디플로이먼트가 생성한 파드 이름은 컨트롤러 객체 이름 + 무작위 문자열이 된다.

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

* 디플로이먼트는 템플릿을 적용해 파드를 생성한다. 템플릿에는 메타데이터 필드로 레이블을 포함한다.
* 디플로이먼트는 레이블 셀렉터를 통해 자신이 관리하는 파드를 식별할 수 있다. 만약 파드의 레이블이 수정되면 디플로이먼트가 해당 파드를 더이상 인식할 수 없게 된다.
* 아래 두 명령을 이용해 디플로이먼트의 상세 정보를 확인하고, 레이블이 일치하는 파드 목록을 조회할 수 있다.

```
kubectl get deploy <deployment name>
```

```
kubectl get pods -l app=<label name>
```

* 아래 명령을 통해 파드의 레이블을 수정할 수 있다.

```
kubectl label pods -l app=hello-kiamol-2 --overwrite app=hello-kiamol-x
```

* 이를 통해 오류가 발생한 파드 레이블을 수정해 운영 환경에서 잠시 제외하고, 디버깅 후 다시 파드 레이블을 원래대로 복구해 디플로이먼트 관리 하에 둘 수 있다.
* 디플로이먼트 리소스 정의에서 포트포워딩 설정으로 트래픽을 허용하려면 아래와 같은 명령을 사용해야 한다. 요청이 들어오면 디플로이먼트가 선택한 파드로 트래픽이 전달된다.

```
kubectl port-forward deploy/<deployment name> <localhost port>:<deploy port>
```

* 더이상 필요 없어진 파드를 직접 삭제하게 되면, 디플로이먼트는 새로운 파드를 다시 띄우게 된다. 따라서 디플로이먼트를 삭제해야 한다.

```
kubectl delete deploy --all
```

## 애플리케이션 매니페스트에 배포 정의하기

* YAML 스크립트는 형상 관리 도구를 사용해 버전 관리를 할 수 있고, 다른 쿠버네티스 클러스터에 동일한 배포가 가능하다는 장점을 갖는다.
* 애플리케이션 매니페스트를 사용하면 직접 명령어를 사용하는 명령형 방식이 아닌 선언적 방식을 통해 최종 결과가 어떻게 되어야 하는지만 명시하게 된다.
* 다음은 간단한 애플리케이션의 매니페스트 스크립트이다. 여기서는 파드를 정의한다.

```yaml
apiVersion: v1
kind: Pod

metadata:
    name: hello-kiamol-3

spec:
    containers:
        - name: web
          image: kiamol/ch02-hello-kiamol
```

* 애플리케이션 매니페스트를 이용해 배포하려면 다음 명령을 사용한다. 원격 URL에 있는 애플리케이션 매니페스트를 통해서도 배포 가능하다.

```
kubectl apply -f pod.yaml

kubectl apply -f https://raw.githubusercontent.com/sixeyed/kiamol/master/ch02/pod.yaml
```

* kubectl apply 명령 시 현재 클러스터에 해당 형상이 실행 중이라면 아무 일도 일어나지 않고, 만약 형상과 다르다면 동일하게 바꾼다.
* 애플리케이션 매니페스트를 통해 디플로이먼트도 정의할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment

# 디플로이먼트의 이름을 정한다.
metadata:
    name: hello-kiamol-4

# 관리 대상을 결정하는 셀렉터를 정의할 수 있다.
# 아래와 같이 특정 레이블과 매치하는 파드만 관리하도록 할 수 있다.
spec:
    selector:
        matchLabels:
            app: hello-kiamol-4

# 디플로이먼트가 파드를 만들 때 추가할 레이블을 지정한다.
template:
    metadata:
        labels:
            app: hello-kiamol-4

# 파드의 정의에 컨테이너 이름과 이미지 이름을 지정한다.
spec:
    containers:
        - name: web
          image: kiamol/ch02-hello-kiamol
```

## 파드에 실행 중인 애플리케이션 접근

* 아래 명령으로 처음 실행한 파드의 내부 IP 주소를 확인할 수 있다.

```
kubectl get pod <podname> -o custom-columns=NAME:metdata,name,POD_IP:status.podIP
```

* 아래 명령으로 파드 내부에 접근해 대화형 셸을 실행할 수 있다.

```
kubectl exec -it <podname> sh
> hostname -i # 파드 내부에서 현재 ip 주소를 확인할 수 있다.
> exit # 셸 세션을 종료한다.
```

* 아래 명령으로 파드의 로그를 확인할 수 있다.

```
kubectl logs --tail=2 <podname> # 파드 이름으로 로그 확인
kubectl logs --tail=2 -l app=hello-kiamol-4 # 레이블에 속한 파드들의 로그 확인
```

* 파드 속의 파일 시스템에 접근할 수 있다. 다음은 파드 내부에 존재하는 파일을 로컬로 복사하는 예제이다.

```
kubectl cp <pod name>:<file path> <local file path>
```
