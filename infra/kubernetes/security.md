# Security

## Network Policy

* 클러스터 내부에서 트래픽을 제어할 수 있다.
* 포트 단위로 파드 사이의 트래픽을 차단하며, 레이블 셀렉터를 통해 식별한다.
*   모든 파드에서 트래픽을 발송하지 못하게 하는 전면 차단 정책을 기반으로 하여, 특정 포트로의 in/out을 허용할 수 있다.

    <figure><img src="../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>
* 네트워크 폴리시 리소스는 인그레스 규칙 형태로 인입 트래픽과 발송되는 트래픽을 제어한다. (인그레스 리소스와는 관련이 없다.)
* 아래는 레이블을 이용해 특정 파드만 접근 가능하도록 하는 네트워크 폴리시 정의이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: apod-api
  labels:
    kiamol: ch16
spec:
  podSelector:
    matchLabels:
      app: apod-api
<strong>  ingress:
</strong><strong>  - from:
</strong><strong>    - podSelector:
</strong><strong>        matchLabels:
</strong><strong>          app: apod-web
</strong><strong>    ports:
</strong><strong>    - port: api # 포트 이름
</strong></code></pre>

* 주의할 점은, 쿠버네티스 네트워크 계층 자체에서 네트워크 폴리시를 지원해야만 사용 가능하다는 것이다. 표준 클러스터 배치에 사용되는 단순 네트워크의 경우 네트워크 폴리시가 지원되지 않으므로 아무리 리소스를 배치해도 적용되지 않는다.
* AKS에서는 클러스터 생성 시 네트워크 폴리시 사용 여부를 지정할 수 있으며, EKS는 클러스터 생성 후 네트워크 플러그인을 직접 교체해야 네트워크 폴리시를 적용할 수 있다.

## Security Context

* 파드와 컨테이너 단위로 보안을 적용할 수 있게 만들어주는 속성이다.
* 컨테이너에 띄워지는 애플리케이션을 루트 권한으로 실행하면 해당 컨테이너에 접근하였을 때 모든 권한을 갖기 때문에 어떠한 정보라도 얻을 수 있게 된다.
* 하지만 간혹 애플리케이션 중 루트 권한으로 실행시켜야만 하는 것들이 있다. 이러한 경우 루트 권한이 아니더라도 실행할 수 있게 수정해야 할 것이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-web
  labels:
    kiamol: ch16
spec:
  selector:
    matchLabels:
      app: pi-web
  template:
    metadata:
      labels:
        app: pi-web
<strong>    spec:
</strong><strong>      securityContext:
</strong><strong>        runAsUser: 65534
</strong><strong>        runAsGroup: 3000
</strong>    # ...
</code></pre>

* 쿠버네티스 API를 사용할 필요가 없는 애플리케이션이 실행되는 컨테이너에는 쿠버네티스 API 토큰이 마운트되는 것을 금지시켜야 한다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-web
  labels:
    kiamol: ch16
spec:
  selector:
    matchLabels:
      app: pi-web
  template:
    metadata:
      labels:
        app: pi-web
    spec:    
<strong>      automountServiceAccountToken: false
</strong>    # ...
</code></pre>

* 컨테이너 정의 시 애플리케이션 프로세스의 권한 상승을 금지하거나 특정 리눅스 커널 기능을 명시적으로 추가하거나 제거할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-web
  labels:
    kiamol: ch16
spec:
  selector:
    matchLabels:
      app: pi-web
  template:
    metadata:
      labels:
        app: pi-web
    spec:    
      automountServiceAccountToken: false
      securityContext:
        runAsUser: 65534
        runAsGroup: 3000
      containers:
        - image: kiamol/ch05-pi
          command: ["dotnet", "Pi.Web.dll", "-m", "web"]
          name: web
          ports:
            - containerPort: 5001
              name: http
          env:
            - name: ASPNETCORE_URLS
              value: http://+:5001
<strong>          securityContext:
</strong><strong>            allowPrivilegeEscalation: false
</strong><strong>            capabilities:
</strong><strong>              drop:
</strong><strong>                - all 
</strong></code></pre>

* 여러 애플리케이션에 일반적으로 적용할 보안 프로파일을 만들어 둘 수도 있다. 이렇게 만들어둔 프로파일이 실제 적용되었는지 검증하려면 admission control 기능을 이용해야 한다.

## Webhook



## Open Policy Agent





## Role-based Access Control

* Role, RoleBinding은 네임스페이스에 속하는 리소스에 대한 권한을 정의한다.
* ClusterRole, ClusterRoleBinding은 특정 네임스페이스에 속하지 않는 리소스에 대한 권한을 정의한다.
* RBAC는 쿠버네티스의 기본 요소가 아니지만 대부분의 플랫폼에서 사용된다.
* 아래 명령으로 RBAC API가 제공되는지 확인할 수 있다.

```
kubectl api-versions | grep rbac
```

* 아래 명령으로 기본 제공되는 클러스터롤과 설명을 조회할 수 있다.

```
kubectl get clusterroles
kubectl describe clusterrole <role name>
```

* 쿠버네티스는 외부 아이덴티티 제공자(active directory, LDAP, OIDC 등)와 결합하여 역할을 관리한다. 즉, 인증 시스템과 통합된 형태가 아니다.
* 사용자용 클라이언트 인증서를 발급하여 API 서버가 인입되는 요청에 인증서를 포함하도록 하면 권한을 제어할 수 있다.
* RBAC의 초기 권한은 아무것도 없고, 허용된 권한을 모두 추가하여 최종 권한 범위가 결정된다. 불허가 권한은 따로 없다.
* 다음 명령은 인증서를 인증 수단으로 등록하고 이를 기반으로 kubectl 컨텍스트를 등록한다.&#x20;

```
kubectl config set-credentials <name> --client-key= --client-certificate= --embed-certs=true

kubectl config set-context <name> --user=<credential name> --cluster <cluster name>
```

* 사용자명과 일치하는 **롤 바인딩을 해주어야 쿠버네티스가 사용자에게 권한을 제공**할 수 있다.
* 다음은 default 네임스페이스에 사용자를 바인딩하는 롤 바인딩 정의이다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: reader-view
  namespace: default
  labels:
    kiamol: ch17
subjects:
- kind: User
  name: reader@kiamol.net
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

* 아래 명령을 통해 특정 네임스페이스의 파드들을 권한을 통해 조회할 수 있다.

```
kubectl get pods -n <namespace name> --as <ssl user credential>
```

* 클러스터 내부 애플리케이션끼리 접근 시에도 보안이 필요하므로 서비스 계정에 대해서는 인증과 권한 부여를 관리한다.

## 서비스 계정

* 서비스 계정은 **쿠버네티스 API 서버를 사용하는 애플리케이션의 보안을 위해 제공**된다. 따라서 애플리케이션 간 리소스 접근 제어하는 것이 목적이 아니다.
* 모든 네임스페이스에는 기본 서비스 계정이 자동으로 생성된다.
* 서비스 계정이 따로 지정되지 않은 파드는 모두 기본 서비스 계정을 사용한다.
* 서비스 계정에 권한을 추가하려면 롤바인딩이나 클러스터 롤바인딩을 만들어야 한다.
* 치명적일 위험이 있는 API를 사용하는 애플리케이션 간에는 각각 전용 서비스 계정을 만드는 것이 좋다.

```
kubectl auth can-i "*" "*" --as system:serviceaccount:<namespace>:default
```

* 다음은 파드 목록 및 상세 정보 조회, 파드 제거 권한을 갖는 서비스 계정을 등록하는 예제이다. 롤과 서비스 계정을 롤 바인딩으로 매핑해주어야 한다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: default-pod-reader
  namespace: default # 롤이 생성되고 적용되는 네임스페이스
  labels:
    kiamol: ch17
rules:
- apiGroups: [""] #core
  resources: ["pods"]
  verbs: ["get", "list", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kube-explorer-default
  namespace: default 
  labels:
    kiamol: ch17
subjects:
- kind: ServiceAccount
  name: kube-explorer
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: default-pod-reader
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-explorer
  labels:
    kiamol: ch17
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-explorer
  labels:
    kiamol: ch17
spec:
  selector:
    matchLabels:
      app: kube-explorer
  template:
    metadata:
      labels:
        app: kube-explorer
    spec:
      serviceAccountName: kube-explorer
      containers:
        - image: kiamol/ch17-kube-explorer
          name: web
          ports:
            - containerPort: 80
              name: http
          env:
          - name: ASPNETCORE_ENVIRONMENT
            value: Development
```

* 각 네임스페이스마다 권한이 다르므로 필요한 롤과 롤바인딩이 각각 존재해야 한다.
* 규칙이 적용되기 전에 대상 리소스가 존재해야 한다. 즉, 네임스페이스 및 서비스 계정이 롤과 롤바인딩 생성 전에 존재해야 한다.
* 만약 kubectl apply 명령으로 디렉토리 하나에 포함된 모든 리소스 정의를 배치하고자 한다면 `02-rbac.yaml` 과 같이 파일 이름 맨앞에 숫자를 붙여 배치 순서를 지정해주어야 한다.

## 계정 그룹

* 사용자 계정과 서비스 계정 모두 그룹에 속할 수 있으며, 그룹을 대상으로 롤바인딩 혹은 클러스터 롤바인딩을 정의할 수 있다.
* 그룹 관리는 인증 시스템에서 수행된다. 인증서를 사용한다면 인증서 발급 및 관리를 담당하는 시스템에서 그룹을 관리해야 한다.
* 쿠버네티스에서는 그룹 단위로 쿠버네티스 접근 권한을 설정한다.
* 서비스 계정은 항상 클러스터 내 모든 서비스 계정 그룹과 네임스페이스 내 모든 서비스 계정 그룹 두 곳에 속한다.
* 하위 리소스를 가진 리소스가 있고 하위 리소스에 접근하려면 별도로 접근 권한을 추가해주어야 한다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: logs-reader
  labels:
    kiamol: ch17
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: test-logs-cluster
  labels:
    kiamol: ch17
subjects:
- kind: Group
  name: test
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: logs-reader
  apiGroup: rbac.authorization.k8s.io
```

* 하나의 네임스페이스에 존재하는 모든 서비스 계정을 한 그룹으로 삼아 롤 바인딩을 할 수 있다.

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kiamol-authn-sre
  labels:
    kiamol: ch17
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sre2
  namespace: kiamol-authn-sre
---
apiVersion: v1
kind: Secret
metadata:
  name: sre2-sa-token
  namespace: kiamol-authn-sre
  annotations:
    kubernetes.io/service-account.name: sre2
type: kubernetes.io/service-account-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sre-sa-view-cluster
  labels:
    kiamol: ch17
subjects:
- kind: Group
  name: system:serviceaccounts:kiamol-authn-sre
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sre-sa-edit-ch17
  namespace: kiamol-ch17
subjects:
- kind: Group
  name: system:serviceaccounts:kiamol-authn-sre
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: edit
  apiGroup: rbac.authorization.k8s.io
```

* 서비스 계정은 인증을 위해 JWT를 사용하며, 이는 kubernetes.io/service-account-token 타입의 비밀값 형태로 파드 볼륨에 저장된다.

## 권한 부여 검증

* kubectl에는 추가 명령을 지원하는 플러그인 시스템이 있어 특정 작업을 수행할 권한이 있는 사용자가 누구인지 파악하거나 각 사용자의 권한 매트릭스가 필요하는 등 확장 기능들을 사용할 수 있다.
* krew 플러그인을 설치하면, 아래와 같이 어떤 사용자 혹은 그룹이 어떤 명령어를 수행할 수 있는지 확인할 수 있다.

```
kubectl krew install who-can
kubectl who-can get configmap todo-web-config
```

* 이밖에도 krew 플러그인에서는 다양한 명령어들을 제공하고 있다.
