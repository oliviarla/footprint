# 2장: 네티의 주요 특징

## 동기와 비동기

* “동기”라는 단어는 업계에 따라 다양한 의미를 갖는다.
* 이 책에서는 함수 또는 서비스의 호출 방식에 관한 내용을 다룬다.

### 동기

* 특정 서비스에 요청을 보내면, 내부에서 모든 작업이 처리 완료된 후에 결과를 받을 수 있다.
* 쉬운 디버깅, 직관적인 흐름 추적이 가능하다.

### 비동기

* 특정 서비스에 요청을 보내면, 내부에서 작업 완료되기 전에 일단 응답을 보낸다.
* 이후 사용자가 다시 결과 확인을 통해 요청을 보내면 어느 과정을 처리하고 있는지 응답을 보내준다.
* Java의 Future 패턴, 이벤트 리스너의 옵저버패턴, Node.js의 콜백 함수, Netty의 Reactor패턴은 모두 비동기 방식이다.
* 수행 시간, 프로그램 구조 등 다양한 고민이 필요하다.
* 네티는 비동기 호출을 위한 API들을 프레임워크 레벨에서 제공하여 스레드 동기화 이슈 및 버그에 대한 부담을 덜어준다.

## 블로킹과 논블로킹

* 소켓의 동작 방식
* 블로킹
  * 요청한 작업이 성공하거나 에러가 발생하기 전까지 응답을 돌려주지 않는 것
* 논블로킹
  * 요청한 작업의 성공 여부와 상관없이 바로 결과를 돌려주는 것
  * 응답값에 의해 에러나 성공 여부를 판단한다.
  * JDK 1.4 부터 Non-Blocking IO를 제공하기 시작했다.
* NIO API를 통해 블로킹과 논블로킹 모드의 소켓을 사용할 수 있다.

### 블로킹 소켓

* ServerSocket을 통해 서버를 연다. 클라이언트가 접속하지 않으면 accept() 메서드 line에서 대기하게 된다.
* 이부분을 수행하는 스레드는 무한 대기(블로킹)하게 된다.
* 클라이언트가 무언가 데이터를 보내지 않으면, InputrStream의 read() 부분에서 또다시 멈추게 된다.
* 블로킹 소켓은 결국 호출된 입출력 메서드의 처리가 완료될 때 까지 응답을 돌려주지 않고 대기한다.

```java
ServerSocket server = new ServerSocket(8888);
System.out.println("접속 대기중");

while (true) {
  Socket sock = server.accept();
  System.out.println("클라이언트 연결됨");

  OutputStream out = sock.getOutputStream();
  InputStream in = sock.getInputStream();

  while (true) {
    try {
        int request = in.read();
        out.write(request);
    }
    catch (IOException e) {
        break;
    }
  }
}
```

* 위 코드 외에도 클라이언트가 소켓 채널에 write할 때 운영체제의 송신 버퍼의 크기가 전송할 데이터를 담을만큼 충분하지 않다면 송신 버퍼가 비워질 때 까지 블로킹된다.
* 블로킹 소켓은 데이터 입출력에서 스레드의 블로킹이 발생하여 동시에 여러 클라이언트를 처리하기에 적합하지 않다.
* 연결 당 스레드를 할당하는 방법
  * 클라이언트 연결과 직결되는 accept() 메서드에서 병목이 발생할 수도 있고, 스레드가 증폭되다가 힙 메모리가 부족해져 OOM 에러가 발생할 수도 있다.
  * OOM 에러를 피하기 위해 스레드 풀을 사용할 수도 있지만 동시 접속 클라이언트 수가 스레드 풀의 스레드 개수에 의존하게 된다.
  * 동시 접속 수를 늘리기 위해 스레드 풀 크기를 자바 힙이 허용하는 최대 크기까지 늘리도록 하더라도, GC가 힙 메모리가 크면 수행시간이 길어져 애플리케이션의 중단 시간이 길어질 수 있고, 수많은 스레드가 CPU 자원을 획득하기 위해 경쟁하며 CPU 자원을 소모하게 된다.

### 논블로킹 소켓

* 소켓에서 데이터를 읽는 read 메서드를 호출했을 때, 블로킹 소켓은 클라이언트가 데이터를 전송해 수신버퍼에 데이터가 들어올 때 까지 블로킹된다.
* 논블로킹 소켓은 클라이언트가 데이터를 전송하지 않았거나 수신 버퍼에 데이터가 들어오지 않았다면 읽어들인 바이트 길이인 0을 돌려준다.
* 다소 복잡한 감이 있지만, 크게 나눠보자면 소켓 서버를 시작하는 startEchoServer 메서드, 연결을 맺는 acceptOP 메서드, I/O로부터 읽는 readOP 메서드, I/O에 쓰는 writeOP 메서드가 있다.

#### startEchoServer

* `Selector` 클래스를 통해 자신에게 등록된 채널에 변경 사항이 발생했는지 검사하고 변경 사항이 발생한 채널에 대한 접근을 가능하게 한다.
* `ServerSocketChannel` 클래스를 통해 소켓 서버를 연다. 소켓 채널을 먼저 생성한 후 포트를 바인딩한다.
* `Selector`객체가 연결 요청인 SelectKey.OP\_ACCEPT 이벤트를 감지하도록 소켓 채널에 등록한다.
* `Selector`에 등록된 채널에서 변경사항이 발생하는지 검사한다. 만약 I/O 이벤트가 발생했다면 어떤 이벤트인지 확인하여 적절한 처리를 위해 메서드를 호출한다.
  * accetable, readable, writable을 확인하고 acceptOp, readOp, writeOp 메서드를 각각 호출하는 부분이 이에 해당한다.

```java
public class NonBlockingServer {
  private Map<SocketChannel, List<byte[]>> keepDataTrack = new HashMap<>();
  private ByteBuffer buffer = ByteBuffer.allocate(2 * 1024);

  private void startEchoServer() {
    try (
        Selector selector = Selector.open();
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()
    ) {

      if ((serverSocketChannel.isOpen()) && (selector.isOpen())) {
        serverSocketChannel.configureBlocking(false);
        serverSocketChannel.bind(new InetSocketAddress(8888));

        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("접속 대기중");

        while (true) {
          selector.select();
          Iterator<SelectionKey> keys = selector.selectedKeys().iterator();

          while (keys.hasNext()) {
            SelectionKey key = (SelectionKey) keys.next();
            keys.remove();

            if (!key.isValid()) {
              continue;
            }

            if (key.isAcceptable()) {
              this.acceptOP(key, selector);
            } else if (key.isReadable()) {
              this.readOP(key);
            } else if (key.isWritable()) {
              this.writeOP(key);
            }
          }
        }
      } else {
        System.out.println("서버 소캣을 생성하지 못했습니다.");
      }
    }
    catch (IOException ex) {
      System.err.println(ex);
    }
  }

  public static void main(String[] args) {
    NonBlockingServer main = new NonBlockingServer();
    main.startEchoServer();
  }
}
```

* 클라이언트의 연결을 수락하고 연결된 소켓 채널을 가져온다.
* 소켓 채널을 논블로킹으로 설정하고 `Selector` 에 등록하여 OP\_READ 이벤트를 감시한다.

```java
private void acceptOP(SelectionKey key, Selector selector) throws IOException {
  ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
  SocketChannel socketChannel = serverChannel.accept();
  socketChannel.configureBlocking(false);

  System.out.println("클라이언트 연결됨 : " + socketChannel.getRemoteAddress());

  keepDataTrack.put(socketChannel, new ArrayList<byte[]>());
  socketChannel.register(selector, SelectionKey.OP_READ);
}
```

*   readOp, writeOp 생략

    * 데이터 읽기, 쓰기 처리하는 메서드는 아래와 같이 구현된다.

    ```java
    private void readOP(SelectionKey key) {
      try {
        SocketChannel socketChannel = (SocketChannel) key.channel();
        buffer.clear();
        int numRead = -1;
        try {
          numRead = socketChannel.read(buffer);
        } catch (IOException e) {
          System.err.println("데이터 읽기 에러!");
        }

        if (numRead == -1) {
          this.keepDataTrack.remove(socketChannel);
          System.out.println("클라이언트 연결 종료 : "
              + socketChannel.getRemoteAddress());
          socketChannel.close();
          key.cancel();
          return;
        }

        byte[] data = new byte[numRead];
        System.arraycopy(buffer.array(), 0, data, 0, numRead);
        System.out.println(new String(data, "UTF-8")
            + " from " + socketChannel.getRemoteAddress());

        doEchoJob(key, data);
      } catch (IOException ex) {
        System.err.println(ex);
      }
    }

    private void doEchoJob(SelectionKey key, byte[] data) {
      SocketChannel socketChannel = (SocketChannel) key.channel();
      List<byte[]> channelData = keepDataTrack.get(socketChannel);
      channelData.add(data);

      key.interestOps(SelectionKey.OP_WRITE);
    }

    private void writeOP(SelectionKey key) throws IOException {
      SocketChannel socketChannel = (SocketChannel) key.channel();

      List<byte[]> channelData = keepDataTrack.get(socketChannel);
      Iterator<byte[]> its = channelData.iterator();

      while (its.hasNext()) {
        byte[] it = its.next();
        its.remove();
        socketChannel.write(ByteBuffer.wrap(it));
      }

      key.interestOps(SelectionKey.OP_READ);
    }
    ```

## 이벤트 기반 프로그래밍

* 이벤트 추상화가 고수준이면 세부적인 제어가 힘들고, 저수준이면 한 동작에 대해 너무 많은 이벤트가 발생해 애플리케이션 성능이 저하된다.
* 서버에 연결된 클라이언트들의 경우 이벤트의 추상화에 대해 깊은 고민이 필요하다. 연결할 클라이언트 수나 이벤트 수가 매우 가변적이며 예측이 불가능하기 때문이다.
* 네트워크 프로그램에서 이벤트가 발생하는 주체는 `소켓` 이다.
  * 소켓이란 데이터 송수신을 위한 네트워크 추상화 단위로, 일반적으로 네트워크 프로그램에서 소켓은 IP와 포트를 가지고 있으며 양방향 네트워크 통신이 가능한 객체이다.
  * 소켓에 데이터를 읽고쓰려면 NIO(소켓 채널) 혹은 스트림(OIO, Old Blocking IO)를 사용해야 한다.
  * 클라이언트 애플리케이션이 소켓에 연결된 스트림에 데이터를 쓰면, 서버에 데이터가 전송된다.
* 발생하는 이벤트는 `소켓 연결`, `데이터 송수신` 으로 나눌 수 있다.
* Netty는 데이터의 읽고 쓰기를 위한 이벤트 핸들러(혹은 데이터 핸들러)인 `ChannelInboundHandlerAdapter` 를 제공한다.
* 사용자는 데이터를 소켓으로부터 직접 읽고 쓰지 않고 이벤트 핸들러를 통해 읽고 쓸 수 있다.
* 서버 애플리케이션의 코드를 클라이언트 애플리케이션에서 재사용할 수 있으며, 각 이벤트에 따라 로직을 분리할 수도 있다.
* 에러 이벤트도 같이 정의하므로, 예외 처리가 쉽다.
