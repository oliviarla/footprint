# Resource Limit

## 리소스 사용량 제한

* 파드 정의에서 컨테이너가 사용 가능한 리소스 총량을 제한할 수 있다. 이를 통해 노드의 CPU, 메모리의 사용량을 특정 파드가 전부 사용해버리지 않도록 할 수 있다.
* 리소스 제한은 컨테이너 수준에서 지정되지만, 파드 정의에 포함되는 내용이므로 kubectl apply로 배치하면 파드가 새 파드로 대체된다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: memory-allocator
  labels:
    kiamol: ch12
spec:
  selector:
    matchLabels:
      app: memory-allocator
  template:
    metadata:
      labels:
        app: memory-allocator
    spec:
      containers:
        - name: api
          image: kiamol/ch12-memory-allocator
<strong>          resources:
</strong><strong>            limits:
</strong><strong>              memory: 50Mi
</strong></code></pre>

* 리소스쿼터 객체를 통해 네임스페이스의 리소스 사용량을 제한할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: ResourceQuota
metadata:
  name: memory-quota
  namespace: kiamol-ch12-memory
spec:
<strong>  hard:
</strong><strong>    limits.memory: 150Mi
</strong></code></pre>
