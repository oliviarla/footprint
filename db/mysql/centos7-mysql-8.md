# 설치 및 연결

## 설치 및 시작과 종료

* MySQL 설치 시 데이터 디렉토리와 로그 파일들을 `/usr/local/mysql` 경로에 저장하고 관리자 모드(root)로 서버 프로세스를 실행한다.
* 설치 시 기본적으로 systemctl 유틸리티를 사용해 서버를 시작/종료할 수 있는 `mysqld.service` 파일을 제공한다.
* my.cnf의 `mysqld_safe` 설정들을 참조하여 서버를 구동해야 하는 경우, systemd를 사용하면 안되고 mysqld\_safe 스크립트를 사용해야 한다.
* 커밋된 데이터를 리두 로그에만 기록해두고 바로 종료시키는 경우, 나중에 다시 시작하더라도 데이터가 반영되어있지 않을 수 있다. 모든 커밋된 데이터를 데이터 파일에 적용하고 종료(클린 셧다운)할 수 있도록 `SET GLOBAL innodb_fast_shutdown=0;` 명령을 미리 적용해둘 수 있다. 이 경우 MySQL 서버가 다시 구동될 때 별도의 트랜잭션 복구 과정을 진행하지 않으므로 빠르게 구동 완료할 수 있다.

## CentOS7에서 MySQL 8 버전 설치하기



<figure><img src="https://blog.kakaocdn.net/dn/Sm5gA/btsbAW1e2q4/zFm0vmFErT9pHBSeSG3KSK/img.png" alt="" width="375"><figcaption></figcaption></figure>

1\. yum을 사용해 install 해주기

```
$ yum install https://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm
$ yum install mysql-server
```

2\. 잘 설치되었는지 확인

```
$ mysql --version
```

3\. mysql을 시작 및 부팅시 자동으로 실행되도록 설정 (안된다면 sudo 권한 사용)

```
systemctl start mysqld
systemctl enable mysqld
```

4\. 초기 비밀번호를 grep 사용해 얻은 후 mysql 접속 (grep 이 안된다면 sudo 권한 사용)

```
$ grep 'temporary password' /var/log/mysqld.log
$ mysql -u root -p
```

5\. 비밀번호 정책 확인 후 조건을 만족하는 비밀번호로 변경

```
$ SHOW VARIABLES LIKE 'validate_password%'
$ alter user 'root'@'localhost' identified by '비밀번호';
```

## 연결

*   MySQL 소켓 파일을 통해 접속하는 경우 Unix domain socket을 사용하게 되며 IPC(Inter Process Communication) 기반으로 통신하게 된다.

    * 별도의 호스트, 포트를 입력하지 않은 경우 기본적으로 소켓 파일을 통해 접속하게 되어있다.

    ```
    mysql -u root -p
    ```
* Host, Port 기반의 TCP/IP 방식으로도 통신할 수도 있다.
