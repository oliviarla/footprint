---
description: 가능하면 데이터를 나눠 보내지 말고 한꺼번에 보내는 알고리즘
---

# 네이글 알고리즘

* TCP/IP 에서 데이터를 전송하려면 헤더를 포함한 패킷을 만들어야 한다.
* 이 헤더의 크기는 50 bytes 정도 되기 때문에 데이터를 나눠 전송하면 불필요한 헤더 정보로 오버헤드가 발생하기 때문에 이를 방지하고자 데이터를 모아서 전송하려는 의도이다.
* 네트워크 상태가 좋지 않고 대역폭이 좁을 때 네트워크를 효율적으로 사용 가능하다.
* 작은 크기의 데이터를 전송하면, 커널의 송신 버퍼에서 적당한 크기가 될 때까지 모아 전송한다.
* 데이터를 송신한 후 ACK 수신하기 전까지 다음 데이터를 전송하지 않는다. 따라서 빠른 응답시간이 필요한 네트워크 애플리케이션에는 좋지 않다.