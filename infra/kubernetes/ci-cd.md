# 개발 워크플로우와 CI/CD

* Dockerfile을 통해 도커 이미지를 빌드하고 이를 kubernetes 리소스 정의에서 활용하면 쉽게 파이프라인을 만들 수 있다.
* 쿠버네티스는 이미지의 태그가 명시적으로 지정되지 않은 경우 `:latest` 태그를 사용하고 이미지를 무조건 내려받기 때문에, 캐시를 사용하려면 명시적으로 태그를 지정해주어야 한다.
* 애플리케이션에 변경 사항이 생겨 도커 이미지를 다시 빌드한 후, `kubectl apply`명령을 사용하더라도 파드 정의는 변경되지 않았으므로 쿠버네티스 상에 자동으로 업데이트가 되지 않는다.
* CI/CD 워크 플로우를 구축하면, 개발자는 로컬 개발 환경을 컨테이너 기술과 별개로 구축해 사용할 수 있다. 컨테이너는 외부 주기에서만 이용된다. 형상 관리 도구에 새로운 코드가 커밋되면 CI/CD에 의해 새로운 버전의 애플리케이션이 테스트용 쿠버네티스 클러스터 환경에 배치되도록 할 수 있다.
* 도커에서 제공하는 빌드킷이라는 도구를 이용하면, 도커가 설치되어 있지 않고 Dockerfile이 없을 때 빌드팩을 이용해 이미지를 빌드할 수 있다. 또한 다른 컴포넌트를 연결할 수 있다.

```bash
buildctl build --frontend=gateway.v0 --opt source=kiamol/buildkit-buildpacks --local context=src --output type=image,name=kiamol/ch11-bulletin-board:buildkit
```

* Github 대신 Gogs라는 Git 서버는 사내용 혹은 백업용으로 사용하기에 적절하다.
* 도커 엔진의 빌드 기능은 완성도가 높고 기존에 많이 사용되고 있기 때문에, 빌드킷이나 깃랩 등 도커를 배제하기 위해 새로운 복잡한 빌드 절차를 도입할 때에는 충분히 여러 사항을 고려해야 한다.
* 쿠버네티스의 네임스페이스를 통해 운영 클러스터를 프로덕트 별로 분할하거나, 개발 클러스터를 개발 팀 기준으로 분할하여 공용 클러스터를 분할해 사용할 수 있다.
* 아래와 같이 yaml 파일에 namespace를 정의할 수도 있고, `kubectl apply -f <파일> --namespace <이름>` 명령을 통해 원하는 namespace 리소스를 배치할 수 있다.

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
<strong>kind: Namespace
</strong><strong>metadata:
</strong><strong>    name: dev-team1
</strong>---
apiVersion: apps/v1
kind: Deployment
<strong>metadata:
</strong><strong>    namespace: dev-team1
</strong>    ...
</code></pre>

* 쿠버네티스의 컨텍스트 전환을 통해 디폴트 네임스페이스가 아닌 사용자 정의 네임스페이스 환경으로 변환할 수 있다.

```bash
kubectl config get-contexts

kubectl config set-context --current --namespace=<이름>
```

* 젠킨스를 이용해 Gogs에 코드 변경이 푸시되면 이를 빌드킷으로 이미지화하고, 해당 이미지를 kubernetes 리소스에서 이용하도록 하여 개발 워크플로우를 구성할 수 있다.
* 이렇게 도커 없이 직접 파이프라인을 구성 할 경우 도커에 대한 학습이 필요 없고 여러 기술을 커스텀하게 구성할 수 있지만, 도커 이미지 최적화나 유연한 도커 기반 워크플로우 구성 등을 할 수 없게 된다.
