---
description: 기존의 I/O API를 대체하기 위해 Java 1.4에 추가된 New I/O
---

# NIO

서버와 클라이언트 간 데이터 송/수신을 위해 채널을 통해서 버퍼에 데이터를 읽고 쓰게 된다.

### 채널

* 파일, 네트워크 소켓 처럼 I/O 작업(읽기 또는 쓰기)을 수행할 수 있는 구성 요소에 대한 연결을 나타내는 타입이다.
* 따라서 여러 종류의 채널을 제공한다.
  * FileChannel: 파일에 데이터를 읽고 쓴다.
  * DatagramChannel: UDP를 이용해 네트워크에서 데이터를 읽고 쓴다.
  * SocketChannel: TCP를 이용해 네트워크에서 데이터를 읽고 쓴다.
  * ServerSocketChannel: 클라이언트의 TCP 연결 요청을 수신(listening)할 수 있으며, SocketChannel은 각 연결마다 생성된다.
* bind(SocketAddress local) 메서드를 통해 자신의 OS 포트에 연결할 수 있다.
* connect(SocketAddress remote) 메서드를 통해 원격 서버와 연결할 수 있다.

### 버퍼

* 고정된 양의 데이터를 담는 타입으로 여러 종류가 제공되어 저장되는 데이터형에 따라 필요한 클래스를 사용할 수 있다.
* 가장 기본적인 버퍼는 ByteBuffer로, 버퍼를 사용해 데이터를 읽고 쓰는 것은 4단계로 진행된다.
  1. 버퍼에 데이터 쓰기
  2. 버퍼의 flip() 메서드 호출
  3. 버퍼에서 데이터 읽기
  4. 버퍼의 clear() 혹은 compact() 메서드 호출
* 채널은 양방향으로 사용하기 때문에 버퍼에 데이터를 쓰다가 읽어야 하는 상황이 온다면 flip() 메서드를 호출해 버퍼의 모드를 쓰기 모드에서 읽기 모드로 변경해야 한다.
* 모든 데이터를 읽은 후에는 버퍼를 지우고 다시 써야 하므로 clear() 메서드를 호출해야 한다.
* 멀티 스레드 환경에서 바이트 버퍼가 공유되지 않도록 주의해야 한다.

#### 버퍼의 속성

* 버퍼는 내부 배열의 상태를 관리하기 위해 아래와 같은 속성들을 가진다.
  * capacity
    * 버퍼에 저장할 수 있는 데이터의 최대 크기로, 한 번 정하면 변경이 불가능하다.
  * position
    * 읽기 또는 쓰기 작업 중인 위치를 나타낸다. 버퍼 객체가 생성될 때 0으로 초기화되고, 데이터를 쓰거나 읽는 메서드가 호출되면 자동으로 증가한다.
  * limit
    * 읽고 쓸 수 있는 버퍼 공간의 최대치로, capacity보다 작은 값이어야 한다.
  * mark

#### 버퍼 생성하기

* 바이트 버퍼를 생성하는 메서드는 아래와 같이 제공된다.
  * allocate
    * JVM 힙 영역에 바이트 버퍼를 생성한다. 메서드의 인수로 바이트 버퍼의 크기를 입력받는다. 최초로 생성된 바이트 버퍼에는 모든 값이 0으로 들어가게 된다.
  * allocateDirect
    * JVM의 힙 영역이 아닌 운영체제의 커널 영역에 바이트 버퍼를 생성한다. ByteBuffer 클래스만 사용할 수 있다. 메서드의 인수로 버퍼의 크기를 입력받으며 최초로 생성된 바이트 버퍼에는 모든 값이 0으로 들어가게 된다.
    * allocate 메서드로 생성된 힙 버퍼에 비해 생성 시간은 길지만 빠른 읽기/쓰기 성능을 보인다.
  * wrap
    * 메서드 인자로 입력된 배열을 사용해 바이트 버퍼를 생성한다. 입력에 사용된 배열이 변경되면 바이트 버퍼의 내용도 변경된다.

#### 버퍼 사용하기

* 데이터를 쓰려면 put 메서드를 사용해 자바의 기본형 데이터를 입력하면 된다.
* 바이트 버퍼에는 capacity만큼의 데이터만 쓸 수 있다. 만약 capacity를 초과하는 데이터를 쓰려고 하면 BufferOverflowException이 발생한다.
* 데이터를 읽으려면 get 메서드를 사용할 수 있다. 다만 데이터를 쓴 이후라면, position 값이 0이 아닌 입력된 데이터의 마지막 인덱스가 되어 있기 때문에, rewind 메서드로 position을 0으로 돌려놓고 읽기 시작해야 한다.
* flip 메서드를 사용하여 이전에 작업한 마지막 위치를 limit 속성으로 변경하고 position을 0으로 변경할 수 있다.

### 셀렉터

* 하나 이상의 채널을 셀렉터에 등록하여 select() 메서드를 호출하면, 등록된 채널 중 이벤트 준비가 완료된 채널이 생길 때까지 블록된다.
* 이를 통해 하나의 스레드에서 여러 채널(소켓 연결)을 관리할 수 있다.

#### 셀렉터 생성하기

```java
Selector selector = Selector.open();
```

#### 셀렉터에 채널 등록하기

* 셀렉터에 채널을 동록하기 위해 반드시 논블로킹 모드로 변경해주어야 한다.
* register() 메서드의 첫번째 인자로 셀렉터를 입력하고, 두번째 인자로 셀렉터를 통해 확인하고자 하는 이벤트의 종류를 전달한다.

```java
ServerSocketChannel channel = ServerSocketChannel.open();
channel.bind(new InetSocketAddress("localhost", 8080));
channel.configureBlocking(false); // 논블로킹 모드로 변경

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

#### SelectionKey

* 채널에서 발생하는 이벤트의 종류를 나타내는 타입이다.
* 이벤트의 종류로는 아래 네 가지 종류가 존재한다.
  * OP\_CONNECT (1 << 0) : 다른 서버에 연결할 준비가 됨
  * OP\_ACCEPT (1 << 2) : 연결 요청을 수락할 준비가 됨
  * OP\_READ (1 << 3) : 데이터를 채널에서 읽어들일 준비가 됨
  * OP\_WRITE (1 << 4) : 채널에 데이터를 쓸 준비가 됨
* interest set을 통해 셀렉터에 등록된 채널이 확인하고자 하는 이벤트의 집합을 확인할 수 있다.
* ready set을 통해 셀렉터에 등록된 채널에서 바로 처리할 수 있도록 이벤트를 준비할 수 있다.

```java
Selector selector = Selector.open();
channel.configureBlocking(false);
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE; 
SelectionKey key = channel.register(selector, interestSet);
```



**출처**

[https://blog.naver.com/beanpole2020/221466876314](https://blog.naver.com/beanpole2020/221466876314)
