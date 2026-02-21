# 기본 명령어

## 파일의 종류

* `일반 파일`
* `디렉토리 파일`
* `링크 파일`
* `실행 파일`
* `압축 파일`

## 환경변수

#### 터미널 시작 시 항상 적용하기

* `~/.bachrc` 파일에 접근해 원하는 환경변수를 터미널 시작할때마다 적용되도록 한다.
* mac 의 경우 \~/.bash-profile 파일에 접근하기

## 디스크

#### 용량 확인

```
df -h
```

```
sudo fdisk -l
```

## 프로세스

#### 포트 번호로 PID 확인하기

```
sudo netstat -nlpn | grep 포트번호
```

```
lsof -i :포트번호
```

#### PID로 실행 위치 찾기

```
ls -al /proc/<pid>
```

## 파일

#### 파일 전송하기

```
# local -> remote
scp <filename> <remote user>@<remote ip>:<remote dir>

scp -r <local dir> <remote user>@<remote ip>:<remote dir>

# remote -> local

scp <remote user>@<remote ip>:<remote dir> <local dir>
```

#### 압축

* 압축하기
  * tar 압축
    * ```
      $ tar -cvf [파일명.tar] [폴더명]

      # abc라는 폴더를 aaa.tar로 압축 예시
      $ tar -cvf aaa.tar abc
      ```
  * tar.gz 압축
    * ```
      $ tar -zcvf [파일명.tar.gz] [폴더명]

      # abc라는 폴더를 aaa.tar.gz로 압축 예시
      $ tar -zcvf aaa.tar.gz abc
      ```
  * zip 압축
    * ```
      $ zip [파일명.zip] [폴더명]

      # 현재폴더 전체를 aaa.zip으로 압축 예시
      $ zip aaa.zip ./*

      # aaa.zip으로 압축하고 현재 폴더의 모든 것과 현재 폴더의 하위 폴더들도 모두 압축 예시
      $ zip aaa.zip -r ./*

      # 위 명령어를 스크립트에서 실행할 때, 파일 경로가 전부 나올 수 있기 때문에 해당 폴더로 이동한 후 작업하는 것을 권장
      ```
* 압축 해제
  * tar 압축 해제
    * ```
      $ tar -xvf [파일명.tar]

      # aaa.tar라는 tar파일 압축해제 예시
      $ tar -xvf aaa.tar
      ```
  * tar.gz 압축 해제
    * ```
      $ tar -zxvf [파일명.tar.gz]

      #  aaa.tar.gz라는 tar.gz파일 압축 해제
      $ tar -zxvf aaa.tar.gz
      ```
  * zip 압축 해제
    * ```
      $ unzip [파일명.zip]

      # aaa.zip 압축 해제 예시
      $ unzip aaa.zip

      # 특정 폴더에 압축해제 예시
      $ unzip aaa.zip -d ./target
      ```
