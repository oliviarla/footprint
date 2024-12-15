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





