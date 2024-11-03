# Network

## 쿠버네티스 내부 네트워크 트래픽 라우팅

* **서비스**란 **파드에 들어오고 나가는 네트워크 트래픽의 라우팅을 맡는 리소스**이다.
* 파드는 쿠버네티스에서 부여한 IP 주소를 가진 가상 환경이며, 다른 컨트롤러 객체에 의해 생애 주기가 결정되는 '쓰고 버리는' 리소스이다.
* 파드의 IP 주소는 파드가 다운되고 새로운 파드가 실행될 때마다 바뀌게 된다.
* 아래와 같이 디플로이먼트로 관리되는 파드를 제거할 경우 기존 구동되었던 파드의 IP와 전혀 다른 IP를 가진 새로운 파드가 실행된다.

```
kubectl get pod -l app=my-pod-2 --output jsonpath='{.items[0].status.podIP}'

kubectl delete pods -l app=my-pod-2

kubectl get pod -l app=my-pod-2 --output jsonpath='{.items[0].status.podIP}'
```

* 쿠버네티스 클러스터에는 전용 DNS 서버가 있어 서비스 이름과 IP 주소를 대응시켜 준다. 즉, 서비스를 생성하면 도메인 이름을 통해 파드에 접근 가능해진다.
* 서비스를 생성하면 서비스의 IP 주소가 클러스터 내 DNS 서버에 등록된다. 서비스가 삭제될 때 까지 IP는 변경되지 않는다.
* 서비스는 복수의 파드가 공유할 수 있는 가상 주소이다.
* 디플로이먼트가 파드와 컨테이너를 추상화한 것 처럼, 서비스는 파드와 파드가 가진 네트워크 주소를 추상화한 것이다.
* 다음은 간단한 서비스 정의 yaml 파일이다. app 레이블을 통해 네트워크 트래픽을 전달할 파드를 지정한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: <서비스이름>
spec:
  selector:
    app: <대상 파드의 app 레이블 값>
  ports:
    - port: 80
```

* 서비스를 배포하려면 kubectl apply 명령을 사용하면 된다.

```
kubectl apply -f <service yml 파일 경로>
```

* 서비스의 상세 정보 출력을 위해서는 아래 명령을 사용하면 된다.

```
kubectl get svc <service 이름>
```

* 아래 명령을 통해 특정 파드에서 다른 파드로 ping 명령을 보낼 순 있지만, 서비스 리소스는 표준 TCP/UDP 프로토콜만 지원하므로 ICMP 프로토콜을 사용하는 ping 명령은 결과적으로 실패하게 된다.

```
kubectl exec <파드이름> -- ping -c 1 <다른 파드이름>
```

* 아래와 같이 파드의 IP를 조회한 후 ping 명령을 보내면 성공할 것이다.

```
kubectl exec <파드이름> -- ping -c 2 $(kubectl get pod -l app=<원하는 파드의 레이블> --output jsonpath='{.items[0].status.podIP}')
```

## 파드와 파드 간 통신

* 클러스터IP는 클러스터 전체에서 통용되는 IP 주소로, 파드가 어느 노드에 있더라도 접근이 가능하다. 클러스터 내에서만 유효하여 파드와 파드 간 통신에서만 사용하다.
* 서비스의 기본 타입은 ClusterIP이므로 type을 따로 명시해주지 않아도 되지만, 의미 상 명시해주는 것이 좋다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: numbers-api
spec:
  ports:
    - port: 80
  selector:
    app: numbers-api
  type: ClusterIP 
```

* 여러 파드 간 통신을 위해 다음과 같이 디플로이먼트를 여러 개 띄울 수 있다. 이 때 numbers/web 파드가 numbers.api 파드로 요청을 보내기 위해 지정한 도메인 이름이 쿠버네티스 내부 서버에 등록되어 있어야 요청이 정상적으로 보내진다.

```bash
kubectl apply -f numbers/api.yaml -f numbers/web.yaml
kubectl wait --for=condition=Ready pod -l app=numbers-web
kubectl port-forward deploy/numbers-web 8080:80
```

* 기존 파드가 내려가고 새로운 파드가 다시 띄워져도 레이블이 일치하며 서비스 IP 주소가 변경되지 않는다.
* 노드가 고장을 일으키는 등의 이유로 파드가 계속해서 교체되어도 별도의 변경 없이 애플리케이션이 서로 통신할 수 있다.

## 외부 트래픽을 파드로 전달하기

### 로드밸런서

* 로드밸런서 서비스는 트래픽을 받은 노드가 아닌 노드에서 실행되는 파드에 트래픽을 전달할 수 있다.
* 레이블 셀렉터와 일치하는 파드로 트래픽을 전달한다.
* 다음은 웹 애플리케이션에 트래픽을 전달하는 서비스의 정의이다. 8080번 포트로 들어오는 트래픽을 파드의 80번 포트에 전달한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: numbers-web
spec:
  ports:
    - port: 8080
      targetPort: 80
  selector:
    app: numbers-web
  type: LoadBalancer
```

* 마찬가지로, kubectl apply 명령을 통해 서비스를 실행시킬 수 있다.
* kubectl port-forward 명령을 각각 설정하는 대신 로드밸런서 서비스를 띄우는 것이 관리하기 편하다.
* 로드밸런서 서비스는 실제 공인 IP 주소를 부여받는다. AKS, EKS 혹은 물리 장비 여러 대에서 로드밸런스 서비스들을 실행시키는 것이 아닌 로컬 도커 데스크탑 혹은 K3S 환경이라면 포트를 각각 다르게 설정해야 한다.

### 노드포트

* 클러스터를 구성하는 모든 노드가 이 서비스에 지정된 포트로 들어온 트래픽을 대상 파드의 대상 포트로 전달한다.
* 외부 로드밸런서를 두지 않으므로 트래픽이 바로 클러스터 노드에 들어가게 된다.
* 트래픽이 클러스터에 들어온 후에는 어느 노드가 요청을 받더라도 대상 파드로 요청이 들어간다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* 서비스에서 설정된 포트가 모든 노드에서 개방되어 있어야 한다. 따라서 로드밸런서 서비스만큼 유연하지 않고, 다중 노드 클러스터에서 로드밸런싱 효과를 얻을 수 없다.
* 또한 클러스터 환경에 따른 동작방식이 다 다르기 때문에 일관성이 없어 실제 사용할 일이 별로 없다.
* 다음은 노드포트의 예제이다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: numbers-web-node
spec:
  ports:
    - port: 8080 # 다른 파드가 서비스에 접근하기 위한 포트
      targetPort: 80 # 대상 파드에 트래픽을 전달하는 포트
      nodePort: 30080 # 서비스가 외부에 공개되는 포트
  selector:
    app: numbers-web
  type: NodePort
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p><a href="https://www.youtube.com/watch?app=desktop&#x26;v=RQbc_Yjb9ls&#x26;ab_channel=AntonPutra">https://www.youtube.com/watch?app=desktop&#x26;v=RQbc_Yjb9ls&#x26;ab_channel=AntonPutra</a></p></figcaption></figure>

## 쿠버네티스 클러스터 외부로 트래픽 전달하기

### ExternalName

* 애플리케이션 파드에서는 로컬 네임을 사용하고, 쿠버네티스 DNS 서버에서 로컬 네임을 조회하면 외부 도메인으로 매핑시켜주는 방식이다.
* external name 서비스는 DNS의 표준 기능 중 하나인 Canonical Name(CNAME)을 사용해 구현되었다.
* 파드는 실제로 클러스터 외부의 컴포넌트와 통신하지만, 내부에서는 로컬 도메인 네임만을 사용하므로 이를 알지 못한다.
* YAML 파일에서 name에는 클러스터 내에서 쓰이는 로컬 도메인 네임을 지정하고, external name에 로컬 도메인 네임에서 포워딩할 외부 도메인을 지정한다.
* 아래와 같이 YAML 파일을 정의하면, 쿠버네티스 DNS 서버에서 도메인 이름으로 numbers-api를 조회하면, raw.githubusercontent.com을 반환한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: numbers-api
spec:
  type: ExternalName
  externalName: raw.githubusercontent.com
```

* 아래 명령을 통해 ExternalName 서비스를 실행시킬 수 있다.

```
kubectl apply -f numbers-services/api-service-externalName.yaml
```

* nslookup 명령으로 서비스의 도메인 이름을 조회해볼 수도 있다.

```
kubectl exec <pod이름> -- sh -c 'nslookup <로컬 도메인 이름> | tail -n 5'
```

* external name 서비스는 애플리케이션이 사용하는 주소가 가리키는 대상만 치환해줄 뿐, 요청 내용을 변경하지는 못한다.
* HTTP 요청이라면 헤더에 대상 호스트 이름이 들어가는데, 헤더의 호스트 이름과 external name 서비스의 응답과 다르면 HTTP 요청이 실패하게 된다.

### Headless

* 로컬 도메인 네임을 IP 주소로 매핑해주는 서비스이다.
* ClusterIP 형태로 정의되지만 레이블 셀렉터가 없어 대상 파드를 정할 수 없다. 대신 제공해야 할 IP 주소 목록이 담긴 엔드포인트 리소스와 함께 배포된다.
* 한 YAML 파일에 여러 리소스를 정의할 때 `---` 을 사용해 구분해야 한다. 다음은 헤드리스 서비스와 엔드포인트를 정의한 YAML 파일이다.

```
apiVersion: v1
kind: Service
metadata:
  name: numbers-api
spec:
  type: ClusterIP
  ports:
    - port: 80
---
kind: Endpoints
apiVersion: v1
metadata:
  name: numbers-api
subsets:
  - addresses:
      - ip: 192.168.123.234
    ports:
      - port: 80
```

* 헤드리스 서비스와 엔드포인트는 kubectl apply 명령으로 실행시킬 수 있다.
* 서비스와 엔드포인트의 상세 정보는 아래 명령을 통해 확인 가능하다.

```
kubectl get svc <서비스 이름>
kubectl get endpoints <엔드포인트 이름>
```

* 헤드리스 서비스를 사용할 때 DNS 조회 결과가 엔트포인트의 IP 주소가 아닌 ClusterIP 주소를 가리킨다.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

* 쿠버네티스 서비스를 --all 을 사용해 지우면 kubernetes API까지 삭제하게 되므로, 서비스를 명시적으로 지정해 삭제해야 한다.

```
kubectl delete svc --all # 권장하지 않음
kubectl delete svc <서비스 이름>
```

## 쿠버네티스 서비스 해소 과정

* 도메인 이름 조회는 쿠버네티스 DNS 서버가 응답한다. 조회 대상이 서비스 리소스이면 클러스터 내 IP 주소 혹은 외부 도메인 이름을 반환한다.
* 파드에서 나온 모든 통신에 대한 라우팅은 네트워크 프록시가 담당한다. 프록시는 각 노드에서 동작하며 모든 서비스의 엔트포인트에 대한 최신 정보를 유지하고, 운영체제가 제공하는 네트워크 패킷 필터(리눅스는 IPVS 또는 iptables)를 사용하여 트래픽을 라우팅한다.
* 파드는 각 노드마다 동작하는 프록시를 거쳐 네트워크에 접근하며, 프록시는 패킷 필터링을 적용해 가상 IP 주소를 실제 엔드포인트로 연결한다.
* 쿠버네티스 DNS 서버는 엔드포인트 IP 주소가 아니라 클러스터의 IP 주소를 반환한다. 엔드포인트가 가리키는 IP 주소는 파드 재구동 시에 변화하기 때문이다.
* 아래는 deployment에 정의된 파드를 임의로 제거하여 자동 재구동 되도록 하고, 엔드포인트의 IP가 변화되었는지 확인할 수 있는 명령이다.

```
kubectl get endpoints <서비스 이름>
kubectl delete pods -l app=<서비스 이름> # 서비스 이름에 속한 파드가 있을 것이다.
kubectl get endpoints <서비스 이름>
```

* 파드 재구동에 영향을 받지 않는 클러스터 IP를 통해 서비스로 오게 되면, 서비스에는 컨트롤러가 있어서 변화하는 엔드포인트를 잘 찾아갈 수 있고, 클라이언트는 DNS 결과를 영구적으로 캐시할 수 있다.

### 네임스페이스

* 모든 쿠버네티스 리소스는 네임스페이스안에 존재한다. 네임스페이스는 여러 리소스를 하나로 묶기 위한 리소스이다.
* 쿠버네티스 클러스터를 논리적인 파티션으로 나누는 역할을 한다.
* 네임스페이스 안에서는 도메인 이름을 이용해 서비스에 접근한다.
* 기본적으로 default 네임 스페이스에 리소스가 속하게 된다.
* DNS 서버나 쿠버네티스 API 같은 쿠버네티스 내부 컴포넌트는 kube-system 네임스페이스에 속한 파드에서 동작한다.
* kubectl에서 `--namespace` 플래그를 사용해 특정 네임스페이스를 대상으로 지정할 수 있다.

```
kubectl get svc --namespace default
kubectl exec deploy/sleep-1 -- sh -c 'nslookup numbers-api.default.svc.cluster.local | grep "^[^*]"'
```

* 네임스페이스를 포함하는 완전한 도메인 이름으로 서비스에 접근할 수 있다.
* 예를 들어, default 네임스페이스에 속한 numbers-api 서비스를 custom 네임스페이스에 속한 파드에서 접근하고자 한다면, `numbers-api.default.svc.cluster.local` 라는 도메인 이름으로 접근할 수 있다.
