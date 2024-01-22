# CentOS7에서 MySQL 8 버전 설치하기

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

&#x20;

&#x20;

&#x20;

출처

[https://engineeringcode.tistory.com/269](https://engineeringcode.tistory.com/269)

[https://kamang-it.tistory.com/entry/MySQL%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C-%EC%A0%95%EC%B1%85-%ED%99%95%EC%9D%B8-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0](https://kamang-it.tistory.com/entry/MySQL%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C-%EC%A0%95%EC%B1%85-%ED%99%95%EC%9D%B8-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0)
