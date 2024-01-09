# Docker

## 특징

* 개발을 위해 MySQL을 사용하려면 맥, 윈도우 환경에 MySQL을 설치해야 하고, 배포를 위해 MySQL을 사용하려면 리눅스 환경에 MySQL을 설치해야 하는 등 각각의 환경에 맞게 설치하는 과정은 번거롭고 에러가 발생하는 경우가 많으며 복잡하다.
* 맥, 윈도우, 리눅스에 Docker만 깔아주면, 그 위에서 MySQL을 실행시키는 방법은 동일하므로 매우 편리하다. (자바의 JVM과 비슷하다고 생각하면 된다.)
* MySQL을 실행하려면 아래 한 줄만 입력하면 된다.

```console
docker run --name mysql_container -e MYSQL_ROOT_PASSWORD=my-password -p 3306:3306 -d mysql:latest
```

* MySQL에 접속하려면 아래 명령어를 입력하면 된다.

```
docker exec -it mysql -u root -p
```

## 용어

### 도커 이미지

* 컨테이너를 만드는 데 사용되는 읽기 전용(Read-only) 템플릿
* 도커 이미지를 생성하려면 컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 도커파일을 만든 후 Dockerfile을 빌드해야 한다.
* 예를 들어 직접 개발한 Spring Boot 어플리케이션을 도커의 컨테이너에서 바로 띄우고 싶다면 Dockerfile을 설정하고 빌드해 이미지를 생성해주어야 한다.

### 도커 컨테이너

* 도커 이미지를 실행한 상태
* 이미지로 컨테이너를 생성하면 이미지의 목적에 맞는 파일이 들어있는 파일 시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성된다.
* 도커 컨테이너는 읽기 전용인 이미지에 변경된 사항을 저장하는 컨테이너 계층(Layer)에 저장한다.

## 기본 명령어

#### docker ps -a

* 현재 실행중인 모든 컨테이너를 조회할 수 있다.

#### docker image ls

* 도커에 다운받은 모든 이미지를 조회할 수 있다.



자세한 명령은 아래 공식 문서를 확인하도록 한다.

{% embed url="https://docs.docker.com/engine/reference/commandline/docker/" %}
