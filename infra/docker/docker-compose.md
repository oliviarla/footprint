# Docker Compose

## &#x20;특징

* 여러 컨테이너 Docker 애플리케이션을 한 번에 정의하고 실행하기 위한 도구이다.
* YAML 파일을 사용하여 애플리케이션 서비스를 구성한 후 해당 파일을 기반으로 여러 컨테이너를 한 번에 생성하고 구동할 수 있다.
* 단일 호스트 장비에 모든 컨테이너를 띄우게 된다.
* 보통 로컬 개발 환경에서 DB 서버, Redis 서버 등을 따로 설치하는 대신 Docker Compose를 통해 한 번에 띄우고 테스트할 수 있어 매우 편리하다.

## 명령어

*   버전 확인

    ```bash
    docker compose version
    ```
*   컨테이너 생성 및 실행

    ```bash
    docker compose up [옵션] [서비스명]
    ```

    | 옵션        | 설명               |
    | --------- | ---------------- |
    | -d        | 백그라운드 실행         |
    | --no-deps | 링크 서비스 실행하지 않음   |
    | --build   | 이미지 빌드           |
    | -t        | 타임아웃을 지정(기본 10초) |

    * &#x20;서비스명에 아무것도 지정하지 않으면 모든 컨테이너가 실행된다.
*   현재 컨테이너 상태 확인

    ```bash
    docker compose ps
    ```
*   컨테이너 로그 출력

    ```bash
    docker compose logs
    ```
*   여러개의 서비스 또는 특정 서비스를 시작 / 정지 / 일시정지 / 재시작을 할 수 있다.

    ```bash
    # 서비스 시작
    docker compose start

    # 서비스 정지
    docker compose stop

    # 서비스 일시 정지
    docker compose pause

    # 서비스 일시 정지 해제
    docker compose unpause

    # 서비스 재시작
    docker compose restart
    ```
*   생성된 컨테이너 일괄 삭제

    ```bash
    docker compose rm
    ```
*   네트워크 정보, 볼륨, 컨테이너들을 일괄 정지 및 삭제 처리

    ```bash
    docker compose down
    ```
*   docker-compose 구성 파일의 내용 확인 (docker-compose.yml의 내용을 출력)

    ```bash
    docker compose config
    ```

## 서비스 실행 순서 제어하기

*   depends\_on

    * 대상 컨테이너가 실행된 후에 현재 컨테이너를 실행하도록 제어할 수 있다.

    ```yaml
    depends_on:
    - container1
    - container2
    ```
*   depends\_on condition&#x20;

    * condition 조건을 사용해 특정 컨테이너가 특정 조건이 되어야 현재 컨테이너를 실행하도록 할 수 있다.

    ```yaml
    depends_on:
      mysql_container:
        condition: service_healthy
        restart: true
    ```

    * `service_healthy`: 의존하는 컨테이너가 healthy 상태가 되어야 컨테이너가 실행되도록 한다.
    * `service_completed_successfully`: 의존하는 컨테이너가 성공적으로 완료되어야 컨테이너가 실행되도록 한다.
