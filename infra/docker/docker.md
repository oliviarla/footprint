# Docker

## 특징

### 실행의 편리함

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

### 의존성 관리의 편리함

* 여러 프로그램을 실행시킬 때 운영체제와 버전 의존성 등을 각 프로그램에 맞추어 조정하기는 쉽지 않다.
* 하나의 프로그램의 버전을 업그레이드할 때 언어나 라이브러리의 버전도 함께 업그레이드해서 사용해야 하는데, 다른 프로그램이 사용중인 의존성이 있다면 쉽게 변경하기 어려울 것이다.
* Docker를 통해 프로그램을 실행시킨다면 이러한 의존성을 각각 갖기 때문에 관리가 편하고, 도커 이미지 자체에 환경을 설정해두기 때문에 프로그램 실행 전에 갖춰야 하는 환경 설정을 하지 않아도 된다.
* 스프링 부트 프로젝트를 도커 이미지로 만들기 위해서는 아래와 같이 Dockerfile을 만들고, docker build 명령을 수행해야 한다. 이렇게 이미지를 만들고 docker run 명령을 수행하면, 자바를 버전에 맞추어 설치할 필요 없이 바로 실행할 수 있다.

```
FROM openjdk:17-alpine
COPY target/your-spring-boot-app.jar /app/your-spring-boot-app.jar
ENTRYPOINT ["java", "-jar", "/app/your-spring-boot-app.jar"]
```

```
docker build -t <docker image name> -f <dockerfile path>

docker run -p 8080:8080 <docker image name>
```

## 원리

* 컨테이너 기술은 예전부터 존재해왔으며, lxc, lxd, lxcfs 등의 종류가 있다. 도커는 LXC 컨테이너를 사용하면서 상위 레벨에서 쉽게 원하는 대로 설정값을 조정할 수 있도록 해준다.
* 컨테이너 간 OS 커널을 공유한다. 이로 인해 특정 OS환경에서 사용되도록 만든 도커 이미지는 특정 OS에서 실행된 도커 호스트에서만 구동 가능하다.
* 예를 들어 Linux환경에 띄워진 도커 호스트에서는 Windows기반 도커 이미지를 구동할 수 없다.

## 용어

### 도커 이미지

* 컨테이너를 만드는 데 사용되는 읽기 전용(Read-only) 템플릿
* 도커 이미지를 생성하려면 컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 도커파일을 만든 후 Dockerfile을 빌드해야 한다.
* 예를 들어 직접 개발한 Spring Boot 어플리케이션을 도커의 컨테이너에서 바로 띄우고 싶다면 Dockerfile을 설정하고 빌드해 이미지를 생성해주어야 한다.

### 도커 컨테이너

* 도커 이미지를 실행하여 생성된 인스턴스
* 이미지로 컨테이너를 생성하면 이미지의 목적에 맞는 파일이 들어있는 파일 시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성된다.
* 도커 컨테이너는 읽기 전용인 이미지에 변경된 사항을 저장하는 컨테이너 계층(Layer)에 저장한다.

## 기본 명령어

* 현재 실행중인 모든 컨테이너 조회

```
docker ps -a
```

* 도커에 다운받은 모든 이미지 조회

```
docker image ls
```

* 사용되지 않는 모든 리소스 제거

```
docker system prune -a
```

자세한 명령은 아래 공식 문서를 확인하도록 한다.

{% embed url="https://docs.docker.com/engine/reference/commandline/docker/" %}
