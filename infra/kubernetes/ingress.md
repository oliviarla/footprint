# Ingress

## 인그레스 라우팅 과정

* 인그레스는 일련의 규칙에 따라 도메인 네임과 애플리케이션의 요청 경로를 매핑해주는 역할을 한다.
* 인그레스는 리버스 프록시를 **인그레스 컨트롤러**로 사용하여 외부 트래픽을 받아들이고 ClusterIP 서비스를 통해 실제 요청을 처리한다. 즉, Nginx나 HAProxy, 컨투어 등을 리버스 프록시로 사용할 수 있다.
* 인그레스 컨트롤러는 파드에서 실행되며 인그레스 객체를 감시하여 변경이 감지되면 해당 규칙을 적용한다.
* 인그레스 객체에는 라우팅 규칙이 어떤 인그레스 컨트롤러이든 사용 가능한 일반적인 형태로 기술되어 있으며, 인그레스 컨트롤러가 이 규칙을 적용해 프록시 역할을 한다.
* 인그레스는 클러스터 전체에 영향을 미치며, 인그레스 라우팅 규칙이 여러 개가 존재할 경우 다른 인그레스 컨트롤러에 영향을 미칠 수 있다. 따라서 도메인 이름을 명시해 라우팅 적용 범위를 최대한 제한하는 것이 좋다.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* 다음은 인그레스 컨트롤러로 nginx를 사용하기 위한 서비스, 디플로이먼트 정의 중 일부이다.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kiamol-ingress-nginx
  labels:
    kiamol: ch15
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: ingress-nginx-controller
  namespace: kiamol-ingress-nginx
data:
  http-snippet: |
    proxy_cache_path /tmp/nginx-cache levels=1:2 keys_zone=static-cache:2m max_size=100m inactive=7d use_temp_path=off;
    proxy_cache_key $scheme$proxy_host$request_uri;
    proxy_cache_lock on;
    proxy_cache_use_stale updating;
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: kiamol-ingress-nginx
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 80
      targetPort: http
    - name: https
      port: 443
      targetPort: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-nginx-controller
  namespace: kiamol-ingress-nginx
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/component: controller
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    spec:
      containers:
        - name: controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0
          args:
            - /nginx-ingress-controller
            - --publish-service=kiamol-ingress-nginx/ingress-nginx-controller
            - --election-id=ingress-controller-leader
            - --ingress-class=nginx
            - --configmap=kiamol-ingress-nginx/ingress-nginx-controller
          securityContext:
            runAsUser: 101
            allowPrivilegeEscalation: true
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - name: https
              containerPort: 443
              protocol: TCP
      serviceAccountName: ingress-nginx
```

* 다음은 인그레스 컨트롤러에 라우팅 규칙을 전달할 인그레스 객체의 정의이다.

```yaml
apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: hello-kiamol
  labels:
    kiamol: ch15
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: hello-kiamol
          servicePort: 80
```

## 인그레스 규칙을 이용한 HTTP 트래픽 라우팅

* 인그레스는 웹 트래픽인 HTTP/HTTPS 요청만 다룬다.
* HTTP 요청에 담긴 라우팅 정보는 크게 호스트와 경로 두 부분으로 나뉜다. 호스트는 도메인 네임(`www.manning.com`)이며, 경로는 자원의 구체적 위치 (`/dotd`)이다.
* 인그레스 객체에서 host 속성을 통해 특정 도메인에 대한 요청에 한해서만 라우팅 규칙을 적용할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: hello-kiamol
  labels:
    kiamol: ch15
spec:
  rules:
<strong>  - host: hello.kiamol.local
</strong>    http:
      paths:
      - path: /
        backend:
          serviceName: hello-kiamol
          servicePort: 80
</code></pre>

* 인그레스를 통해 각 경로에 따라 서로 다른 서비스에 요청을 전달할 수 있다.
* 컨트롤러로부터 입력받은 URL 대신 다른 URL로 서비스 요청을 보내려면, rewriting 관련 어노테이션에서 정규 표현식을 이용해 요청 경로를 대상 경로로 변환해야 한다.
* 아래 예제에서는 vweb.kiamol.local 도메인으로 들어온 `/` , `/v1` 요청을 각각 다른 서비스를 이용해 처리하며, rewrite-target 어노테이션을 이용해 경로를 모두 `/`로  변환한다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: vweb
  labels:
    kiamol: ch15
  annotations:
<strong>    nginx.ingress.kubernetes.io/rewrite-target: /
</strong>spec:
  rules:
  - host: vweb.kiamol.local
    http:
      paths:
      - path: /
        backend:
          serviceName: vweb-v2
          servicePort: 80
      - path: /v1
        backend:
          serviceName: vweb-v1
          servicePort: 80
      - path: /v2
        backend:
          serviceName: vweb-v2
          servicePort: 80
</code></pre>

* 인그레스 규칙에 모니터링 지표를 위한 엔드포인트 등 외부로 노출되면 안되는 경로를 정의하지 않는다면 외부로부터의 접근이 차단된다. 하지만 공개하기 위한 경로를 모두 열거해야 하기 때문에 인그레스 정의 파일이 복잡해질 수는 있다.
* pathType 속성을 통해 인그레스 경로가 완벽히 일치해야 하는지, 처음 일부만 일치해야 하는지 설정할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: todo
  labels:
    kiamol: ch15
spec:
  rules:
  - host: todo.kiamol.local
    http:
      paths:
<strong>      - pathType: Exact
</strong>        path: /
        backend:
          serviceName: todo-web
          servicePort: 80 
<strong>      - pathType: Prefix
</strong>        path: /static
        backend:
          serviceName: todo-web
          servicePort: 80
</code></pre>

## 인그레스 컨트롤러

* 인그레스 컨트롤러의 유형은 다음 두 가지가 있다.
  * 리버스 프록시는 네트워크 수준에서 동작하고 호스트네임을 기준으로 콘텐츠를 가져온다.
  * 현대적 프록시는 경량화되었고 플랫폼마다 다르게 동작하며 다른 서비스와 통합이 용이하다.

### 다양한 기능 제공

* 인그레스 컨트롤러는 SSL 터미네이션, 방화벽, 캐싱, 단위 시간 당 접근 수 제한, URL rewriting, 허용 클라이언트 IP 목록 등의 기능을 지원한다.
* 다음은 캐싱을 적용한 인그레스 정의의 예시이다. 캐싱 기능은 인그레스 규격에서 정의된 것이 아니라 지원하지 않는 인그레스 컨트롤러도 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: pi
  labels:
    kiamol: ch15
  annotations:
<strong>    nginx.ingress.kubernetes.io/proxy-buffering: "on" 
</strong><strong>    nginx.ingress.kubernetes.io/configuration-snippet: |
</strong><strong>      proxy_cache static-cache;
</strong><strong>      proxy_cache_valid 10m;
</strong>spec:
  rules:
  - host: pi.kiamol.local
    http:
      paths:
      - path: /
        backend:
          serviceName: pi-web
          servicePort: 80
</code></pre>

* 한 사용자가 보낸 요청들은 동일한 컨테이너로만 전달하는 sticky session 기능역시 인그레스 정의에 추가해 사용할 수 있다.
  * 크로스 사이트 요청 위조 방지 대책으로 인해 데이터의 값을 변경하는 요청들에 한해 서로 다른 파드에 요청이 분배되면 400 Bad Request 응답이 올 수 있다. 따라서 데이터의 값을 변경하는 요청들에 한해 sticky session을 적용해 로드밸런싱을 해제해주어야 한다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: todo-new
  annotations:
<strong>    nginx.ingress.kubernetes.io/affinity: cookie
</strong>spec:
  rules:
  - host: todo.kiamol.local
    http:
      paths:
      - pathType: Exact
        path: /new
        backend:
          serviceName: todo-web
          servicePort: 80
</code></pre>

* 인그레스의 캐싱, Sticky Session 등의 기능은 모든 경로 설정에 적용되기 때문에 서로 다른 기능을 사용하고자 한다면 인그레스를 여러개 두어야 한다.

### 여러 인그레스 컨트롤러 사용

* 한 클러스터 안에 여러 개의 인그레스 컨트롤러를 두면 로드밸런서 서비스 형태로 외부에 노출된다. 각 인그레스 컨트롤러들이 각각 IP주소를 갖도록 하고 도메인 네임을 인그레스에 연결할 수 있다.
* 인그레스 클래스를 이용해 애플리케이션에서 어떤 인그레스 컨트롤러를 사용할 지 정할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: Service
metadata:
  name: todo-web-sticky
  labels:
    kiamol: ch15
  annotations:
<strong>    kubernetes.io/ingress.class: traefik
</strong>    traefik.ingress.kubernetes.io/service.sticky.cookie: "true"
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: todo-web
---
apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: todo2-new
  labels:
    kiamol: ch15
  annotations:
<strong>    kubernetes.io/ingress.class: traefik
</strong>    traefik.ingress.kubernetes.io/router.pathmatcher: Path
spec:
  rules:
  - host: todo2.kiamol.local
    http:
      paths:
      - path: /new
        backend:
          serviceName: todo-web-sticky
          servicePort: 80
</code></pre>

## 인그레스로 HTTPS 적용

* 인그레스 컨트롤러에서 HTTPS를 지원하기 위한 TLS 인증서를 비밀값에서 불러올 수 있다. 이를 통해 각 애플리케이션에서 사용할 인증서를 한 곳에서 관리할 수 있다는 장점을 가진다.

```
kubectl create secret tls <비밀값이름> --key=<key파일> --cert=<crt파일>
```

<pre class="language-yaml"><code class="lang-yaml">apiVersion: networking.k8s.io/v1beta1 
kind: Ingress
metadata:
  name: todo2-new
  labels:
    kiamol: ch15
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/router.pathmatcher: Path
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  rules:
  - host: todo2.kiamol.local
    http:
      paths:
      - path: /new
        backend:
          serviceName: todo-web-sticky
          servicePort: 80
<strong>  tls:
</strong><strong>   - secretName: &#x3C;비밀값이름>
</strong></code></pre>

* Traefik 인그레스 컨트롤러를 사용하면 Traefik을 배치할 때 생성되는 자체 서명된 인증서를 사용할 수 있으며, 어노테이션에서 TLS와 기본 TLS 리졸버를 활성화할 수 있다. 단, 자체 서명 인증서를 사용하면 브라우저에서 완전히 안전한 상태가 아니라는 경고가 뜬다.

## 인그레스의 단점

* 현재는 인그레스 컨트롤러가 제공하는 기능과 어노테이션이 각각 다르기 때문에 여러 인그레스 컨트롤러를 사용할 경우 어려움을 겪을 수 있다.
* 다양한 배포 전략을 이용한 트래픽 제어 기능을 지원하지 않으며 서비스의 회복 탄력성 관련 기능(retry, circuit breaker 등)이 없다. 이러한 인그레스의 한계점은 Istio를 통해 극복할 수 있다.
