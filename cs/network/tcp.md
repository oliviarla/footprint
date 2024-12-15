# TCP 네트워크

## 3-Way Handshake

* 클라이언트가 서버로 SYN 패킷을 송신하면, 서버는 SYN-ACK 패킷을 클라이언트에 전송한다. 이 때 서버 소켓의 상태는 SYN\_RECEIVED 이다.
*   SYN-ACK 패킷을 수신한 클라이언트는 서버로 ACK 패킷을 보낸다.

    <figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 소켓 종료

* 종료를 원하는 Initiator에서 socket.close()와 같은 소켓 종료함수를 호출한다.
* 해당 소켓에 대한 권한이 tcp커널로 넘어가게 되고, 만약 소켓이 블로킹 소켓이면 위의 종료과정이 완전히 끝날때까지 블로킹되고, 논블로킹 소켓이면 EWOULDBLOCK을 리턴한다.
* Initiator는 FIN\_WAIT상태에 들어가고 상대호스트의 FIN 패킷을 기다린다.
* FIN 메시지를 받은 Receiver는 받은 메시지에 대한 ACK 신호를 보내고 소켓 종료 함수를 호출해 FIN 패킷을 보내기 전까지 ClOSE\_WAIT상태가 된다.
*   Receiver가 종료 함수를 호출하여 FIN을 보내면 Initiator는 받은 FIN에 대한 ACK를 보낸 후 일정 시간 동안 TIME\_WAIT 상태가 된다.

    > TIME\_WAIT은 자신이 보낸 ACK가 잘 도착했는지 확인 하기 위해 기다렸다 종료하는 유예 시간이다.\
    > 리눅스의 경우 90초 정도라고 하며, 자신이 보낸 ACK가 손실됬다면 상대는 다시 FIN을 보내게 되고 그에 따른 ACK를 보낼 수 있다.

    &#x20;
*   만약 TIME\_WAIT 상태인 포트로 접속하기 위해선 TIME\_WAIT이 끝날때까지 기다려야 한다. 따라서 고정적인 포트에 바인딩하는 서버의 경우 에러가 나서 먼저 FIN을 보내고 TIME\_WAIT에 걸렸을시 곧바로 서버를 재가동 시키지 못하는 불상사가 발생한다.

    &#x20;

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>





**출처**

[https://dhdnjswo5000.tistory.com/40](https://dhdnjswo5000.tistory.com/40)
