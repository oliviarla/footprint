# CRD & Operator

## CRD

* Custom Resource Definition의 약자로, 쿠버네티스에 새로운 리소스 타입을 추가할 때 사용된다.
* 다른 쿠버네티스 객체들과 동일하게 처리된다.
* CRD 정보는 etcd 데이터베이스에 저장된다.
* API의 표준 액션인 create, get, list, watch, delete 등을 모두 지원한다.

### 사용법

* 쿠버네티스 리소스의 구조를 기술하는 스키마를 CRD 객체로 생성한다.
* 다음은 todo 항목을 저장하는 간단한 쿠버네티스 객체를 정의하는 CRD 예시이다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: apiextensions.k8s.io/v1  # minimum 1.16
kind: CustomResourceDefinition
metadata:
  name: todos.ch20.kiamol.net
  labels:
    kiamol: ch20
spec:
  group: ch20.kiamol.net
  scope: Namespaced
  names:
<strong>    plural: todos
</strong><strong>    singular: todo
</strong><strong>    kind: ToDo
</strong>    shortNames:
    - td
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
<strong>                item:
</strong>                  type: string
<strong>                dueDate:
</strong>                  type: string
                  format: date 
</code></pre>

* 다음은 위 CRD 스키마에 따라 정의된 CRD 리소스의 객체 예시이다. apiVersion과 리소스 유형이 CRD와 일치해야 한다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: "ch20.kiamol.net/v1"
kind: ToDo
metadata:
  name: ch20
  labels:
    kiamol: ch20
spec:
<strong>  item: "Finish KIAMOL Ch20"
</strong><strong>  dueDate: "2020-07-26"
</strong></code></pre>

* `kubectl get todos` 명령으로 현재 생성된 ToDo 리소스 객체들 목록을 확인할 수 있고, `kubectl delete todo ch20` 명령으로 커스텀 객체를 제거할 수 있다.
* 이러한 형태는 예시일 뿐이고 실제로 CRD를 사용할 때에는 리소스나 클러스터 기능을 확장하는 용도로 사용해야 한다.
* `kubectl delete crd <crd name>` 명령으로 CRD 자체를 삭제하면 모든 객체가 사라지므로 CRD 리소스에 대한 RBAC 권한을 엄격히 관리해야 한다.
* 사용자 정의 컨트롤러와 함께 사용하여 CRD 객체를 관리하도록 할 수 있다. 하지만 관리자가 특정 CRD 객체를 직접 관리하거나 삭제해도 문제가 없도록 컨트롤러 로직을 구성해야 한다. 예를 들어 CRD 객체의 네임스페이스, 비밀값 등이 사라졌다면 다시 생성해주어야 한다.

## 오퍼레이터

* CRD 및 컨트롤러를 사용해 애플리케이션에 완전한 생애 주기 관리를 제공한다.
* 데이터베이스를 업그레이드할 때 읽기 전용으로 변경하고 백업한 후 업그레이드해야 할 때 내장된 쿠버네티스 리소스나 헬름을 이용해서 구현하는 것은 어렵다. 오퍼레이터를 이용하면 이러한 작업들을 구현해두고 추상화시킬 수 있다.
* 오퍼레이터를 배치하면 서드파티 컴포넌트를 하나의 서비스처럼 사용할 수 있도록 관리해준다.
* 이 때 파드는 디폴로이먼트나 스테이트풀셋 대신 오퍼레이터가 컨트롤러 역할을 하여 관리한다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: todo-db-secret
  labels:
    kiamol: ch20
type: Opaque
stringData:
  ROOT_PASSWORD: "kiamol-2*2*"
---
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: todo-db
spec:
  mysqlVersion: "5.7.24"
  replicas: 2
  secretName: todo-db-secret  
  podSpec:    
    resources:
      limits:
        memory: 200Mi
        cpu: 200m
```



