# Linux에서 root 아닌 유저로 docker 실행하기

linux에서 root 권한이 아닌 상태로 docker를 실행하면 권한 문제가 발생할 수 있다.

```
[mkkim@localhost ~]$ docker service ls
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/services?status=true": dial unix /var/run/docker.sock: connect: permission denied
```

이런 경우 docker group에 해당 유저를 추가해주어야한다.

1. 보통은 docker group이 생겼을테지만, 만약 없으면 생성해준다.

```
sudo groupadd docker
```

2. docker group에 현재 접속중인 유저를 추가한다.

```
sudo usermod -aG docker $USER
```

3. &#x20;재 연결 후 다시 명령어를 입력해보면 권한 문제가 발생하지 않고 정상 동작함을 확인할 수 있다.

```
[mkkim@localhost ~]$ docker service ls
Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```

출처

[https://seulcode.tistory.com/557](https://seulcode.tistory.com/557)
