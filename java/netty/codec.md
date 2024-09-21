---
description: 이벤트 핸들러를 상속받아 구현한 코덱에 대해 알아본다.
---

# 코덱

## 개념

* 모든 네트워크 애플리케이션은 피어 간 전송되는 바이트를 프로그램의 데이터 포맷으로 변환하는 방법을 정의해야 한다.
* **바이트와 프로그램의 데이터 포맷 간 변환**은 인코더와 디코더로 구성된 **코덱에 의해 처리**된다.
* 네티의 와 ChannelOutboundHandler는 각각 인코더와 디코더 역할을 한다.
* 인코더는 특정 애플리케이션에서 의미가 있는 바이트의 시퀀스 구조인 메시지를 전송에 적합한 바이트 스트림 형태로 변환한다. 따라서 아웃바운드 데이터를 처리한다.
* 디코더는 네트워크 스트림을 프로그램의 메시지로 변환한다. 따라서 인바운드 데이터를 처리한다.



## 디코더

* 디코더는 인바운드 데이터를 다른 포맷으로 변환하므로, ChannelInboundHandler의 구현체여야 한다.
* 인바운드 데이터를 변환하여 ChannelPipeline 내에서 다음 ChannelInboundHandler로 전달하기 위해 사용된다.
* 여러 디코더를 체인으로 연결하여 복잡한 변환 논리도 나누어 처리할 수 있다. 이 덕분에 코드 모듈성과 재사용성을 지킬 수 있다.

### ByteToMessageDecoder

* 네티는 바이트 스트림을 메시지로 디코딩하는 작업이 매우 빈번하므로 이를 위한 추상 클래스를 제공한다.
* 인바운드 데이터가 처리할 만큼 모일 때까지 버퍼에 저장한다.
* `decode 추상 메서드`를 제공하므로 ByteBuf에 있는 데이터를 List\<Object>에 추가하는 로직을 작성해야 한다.
* Channel이 비활성화될 때 호출될 `decodeLast 메서드`를 제공하며, 단순히 decode 메서드를 호출하고 있어 특별한 처리가 필요하면 재정의해야 한다.
* 단순한 int를 포함하는 바이트 스트림을 처리하는 ToIntegerDecoder의 예제는 아래와 같다.
  * 인바운드 데이터를 4 바이트씩 읽고 int로 변환한 후 out 리스트에 추가한다.

```java
public class ToIntegerDecoder extends ByteToMessageDecoder {
    @Override
    public void decode(ChannelHandlerContext ctx, ByteBuf in,
        List<Object> out) throws Exception {
        if (in.readableBytes() >= 4) {
            out.add(in.readInt());
        }
    }
}
```

> 참조 카운팅 시 메시지가 인코딩/디코딩되면 ReferenceCountUtil.release(message) 호출을 통해 자동으로 해제된다. 참조를 나중에 이용하기 위해 유지하려면 ReferenceCountUtil.retain(message) 를 호출하여 참조 카운트를 증가시켜 메시지가 해제되지 않도록 한다.

### ReplayingDecoder

* ByteToMessageDecoder를 확장하는 추상 클래스이며, readableBytes()를 호출하지 않아도 되도록 내부적으로 처리해준다.
* ByteToMessageDecoder에 비해 약간 느리다.
* 복잡한 처리를 해야 하는 상황일 경우 이 클래스를 사용하는 것이 좋다.
* 이를 확장한 ToIntegerDecoder의 예제는 아래와 같다.
  * decode 메서드의 ByteBuf 타입의 인자는 ReplayingDecoderBuffer 타입으로 들어오며, readInt 메서드에서 읽을 바이트가 충분하지 않으면 예외가 발생할 것이다.&#x20;

```java
public class ToIntegerDecoder extends ReplayingDecoder<Void> {

    @Override
    public void decode(ChannelHandlerContext ctx, ByteBuf in,
        List<Object> out) throws Exception {
        out.add(in.readInt());
    }
}
```

### MessageToMessageDecoder

* 특정 메시지 타입을 다른 타입으로 변환할 때 사용하는 추상 클래스이다.
* 제네릭에 입력 타입을 명시한 후 상속받아 decode 메서드가 입력 타입을 인자로 받도록 한다.
* IntegerToStringDecoder의 예제는 아래와 같다.
  * 디코딩된 String 타입 객체는 리스트에 넣어 다음 Handler가 사용할 수 있도록 한다.

```java
public class IntegerToStringDecoder extends MessageToMessageDecoder<Integer> {
    @Override
    public void decode(ChannelHandlerContext ctx, Integer msg,
        List<Object> out) throws Exception {
        out.add(String.valueOf(msg));
    }
}
```

### TooLongFrameException

* 프레임이 지정한 크기를 초과하면 발생하는 예외
* 디코딩 가능할 때 까지 바이트를 메모리 버퍼에 저장해야 하며, 디코더가 메모리를 소진할 만큼 많은 데이터를 저장하지 않게 하기 위해 적절한 최대 바이트를 지정해야 한다.
* 이렇게 던져진 예외는 ChannelHandler.exceptionCaught() 메서드를 통해 catch하여 처리할 수 있다.

```java
public class SafeByteToMessageDecoder extends ByteToMessageDecoder {
    private static final int MAX_FRAME_SIZE = 1024;
    @Override
    public void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        int readable = in.readableBytes();
        if (readable > MAX_FRAME_SIZE) {
            in.skipBytes(readable);
            throw new TooLongFrameException("Frame too big!");
        }    
    }
}
```

## 인코더

* 인코더 클래스의 경우 ChannelOutboundHandlerAdapter를 상속받은 추상 클래스인 MessageToMessageEncoder를 상속받아 encode 메서드를 구현하는 템플릿 메서드 패턴으로 구현된다. 이를 통해 다양한 종류의 인코더/디코더를 제공할 수 있다.
* 아래와 같이 write 이벤트가 발생할 경우 ChannelOutboundHandler를 구현한 인코더에서 메시지를 바이트나 다른 메시지로 변환해주게 된다.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>

### MessageToByteEncoder

* encode 추상 메서드를 제공하여, ByteBuf로 인코딩할 아웃바운드 메시지와 ByteBuf 객체를 입력받아 인코딩된 결과를 ByteBuf 객체에 대입하는 로직을 작성해야 한다.
* 아래는 Short 객체를 입력받아 바이트로 변환하는 인코더의 예제이다.

```java
public class ShortToByteEncoder extends MessageToByteEncoder<Short> {
    @Override
    public void encode(ChannelHandlerContext ctx, Short msg, ByteBuf out) throws Exception {
        out.writeShort(msg);
    }
}
```

### MessageToMessageEncoder

* 아웃바운드 데이터를 특정 타입으로 인코딩하는 인코더를 위한 클래스이다.

```java
public class IntegerToStringEncoder extends MessageToMessageEncoder<Integer> {
    @Override
    public void encode(ChannelHandlerContext ctx, Integer msg,
        List<Object> out) throws Exception {
        out.add(String.valueOf(msg));
    }
}
```

## 추상 코덱 클래스

* 디코더/인코더를 묶어 처리한다.
* 이를 위해 ChannelInboundHandler, ChannelOutboundHandler를 모두 구현한다.

### ByteToMessageCodec

* 모든 종류의 요청/응답 프로토콜에 활용 가능하다.
* 코덱이 들어오는 바이트를 읽고 프로그램에서 사용하는 메시지 타입(ex. SmtpRequest 객체)으로 디코딩할 수 있다.
* 응답을 생성할 때 메시지 타입(ex. SmtpResponse 객체)을 바이트로 인코딩할 수 있다.

### MessageToMessageCodec

* INBOUND\_IN, OUTBOUND\_IN 제네릭 타입을 두어 decode 메서드는 INBOUND\_IN를 OUTBOUND\_IN로 디코딩하고, encode 메서드는 OUTBOUND\_IN을 INBOUND\_IN로 인코딩하도록 한다.
* 레거시 메시지 포맷이나 특정 기업의 메시지 포맷을 이용하는 API와 상호운용할 때 자주 사용된다.

### CombinedChannelDuplexHandler

* 디코더/인코더를 결합하여 재사용성이 저하되는 것을 방지할 수 있는 클래스이다.
* 디코더 클래스와 인코더 클래스를 확장한 제네릭 타입을 가짐으로써 추상 코덱 클래스를 직접 확장하지 않고도 코덱을 구현할 수 있다.
* 직접 정의한 ByteToCharDecoder, CharToByteEncoder 타입을 제네릭 타입으로 두고 해당 타입의 객체를 생성해 Handler 내부에서 사용되도록 한다.

```java
public class CombinedByteCharCodec extends
    CombinedChannelDuplexHandler<ByteToCharDecoder, CharToByteEncoder> {
    public CombinedByteCharCodec() {
        super(new ByteToCharDecoder(), new CharToByteEncoder());
    }
}
```

## 코덱의 종류

* base64 코덱
  * 8bit 이진 데이터를 문자 코드에 영향받지 않는 공통 ASCII 영역의 문자로 이뤄진 문자열로 변환하는 base64 인코딩을 지원한다.
* bytes 코덱
  * 바이트 배열 데이터에 대한 송수신을 지원한다.
* compression 코덱
  * 송수신 데이터의 압축을 지원하며, 다양한 압축 알고리즘을 지원한다.
* http 코덱
  * HTTP 프로토콜을 지원하며, 세부 구현체로 cors 코덱, multipart 코덱, websocketx 코덱을 지원한다.
* marshalling 코덱
  * 네트워크를 통해 송/수신 가능한 형태로 변환하는 JBoss의 marshalling 라이브러리를 지원한다.
  * JBoss marshalling 라이브러리는 기존 JDK의 직렬화/역직렬화 문제점을 해결하기 위해 고안되었다.
* protobuf 코덱
  * 구글의 프로토콜 버퍼를 사용한 데이터 송수신을 지원한다.
* rtsp 코덱
  * 실시간 데이터 스트리밍을 위해 만들어진 애플리케이션 레벨의 rtsp 프로토콜을 지원한다.
* sctp 코덱
  * sctp 전송 계층을 사용하도록 하는 코덱이며, 부트스트랩의 채널을 NioSctpChannel 혹은 NioSctpServerChannel로 설정해야 한다.
* string 코덱
  * 문자열의 송수신을 지원하여, 텔넷이나 채팅 서버의 프로토콜에 이용된다.
* serialization 코덱
  * 자바의 객체를 직렬화/역직렬화 할 수 있도록 지원하는 코덱이며, JDK의 ObjectOutputStream/ObjectInputStream과 호환되지 않는다.

## 사용자 정의 코덱

* 사용자가 직접 필요한 프로토콜을 구현해 사용할 수 있다.
* 아래는 간단한 Http 웹서버 예제를 구현하는 방법이다.

**1) 서버 구동을 위한 부트스트랩 작성**

```java
public final class HttpHelloWorldServer {

    static final boolean SSL = System.getProperty("ssl") != null;
    static final int PORT = Integer.parseInt(System.getProperty("port", SSL? "8443" : "8080"));

    public static void main(String[] args) throws Exception {
        // Configure SSL.
        final SslContext sslCtx = ServerUtil.buildSslContext();

        // Configure the server.
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.option(ChannelOption.SO_BACKLOG, 1024);
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class)
             .handler(new LoggingHandler(LogLevel.INFO))
             .childHandler(new HttpHelloWorldServerInitializer(sslCtx));

            Channel ch = b.bind(PORT).sync().channel();

            System.err.println("Open your web browser and navigate to " +
                    (SSL? "https" : "http") + "://127.0.0.1:" + PORT + '/');

            ch.closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

**2) 이벤트 핸들러 작성**

* Bootstrap의 childHandler 메서드에 이벤트 핸들러를 등록하기 위해 ChannelInitializer를 아래와 같이 구현한다.
* `HttpServerCodec`과 직접 정의할 사용자 코덱인 `HttpHelloWorldServerHandler`를 핸들러로 추가한다.

> HttpServerCodec
>
> * 간단한 웹 서버를 생성하는 데 사용하는 코덱
> * HttpRequestDecoder
>   * ByteBuf 객체를 HttpRequest와 HttpContent로 디코딩해준다.
> * HttpResponseEncoder
>   * HttpResponse를 ByteBuf 로 인코딩해준다.

<pre class="language-java"><code class="lang-java">public class HttpHelloWorldServerInitializer extends ChannelInitializer&#x3C;SocketChannel> {

    private final SslContext sslCtx;

    public HttpHelloWorldServerInitializer(SslContext sslCtx) {
        this.sslCtx = sslCtx;
    }

    @Override
    public void initChannel(SocketChannel ch) {
        ChannelPipeline p = ch.pipeline();
        if (sslCtx != null) {
            p.addLast(sslCtx.newHandler(ch.alloc()));
        }
<strong>        p.addLast(new HttpServerCodec());
</strong>        p.addLast(new HttpContentCompressor((CompressionOptions[]) null));
        p.addLast(new HttpServerExpectContinueHandler());
<strong>        p.addLast(new HttpHelloWorldServerHandler());
</strong>    }
}
</code></pre>

**3) 사용자 정의 코덱 작성**

* 아래는 사용자 정의 코덱인 `HttpHelloWorldServerHandler` 이다.
* ChannelInboundHandler를 구현하기 때문에, channelRead 이벤트로 수신되는 HttpRequest, HttpMessage, LastHttpContent 객체를 처리할 수 있다. 각 객체에 대한 자세한 내용은 9장에 나온다.
* channelReadComplete 메서드를 통해 웹브라우저로부터 데이터가 모두 수신되었을 때 채널 버퍼의 내용을 웹 브라우저에 전달한다.
* channelRead0 메서드를 통해 content가 "Hello World"인 HttpResponse를 채널에 쓴다.

```java
public class HttpHelloWorldServerHandler extends SimpleChannelInboundHandler<HttpObject> {
    private static final byte[] CONTENT = { 'H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd' };

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.flush();
    }

    @Override
    public void channelRead0(ChannelHandlerContext ctx, HttpObject msg) {
        if (msg instanceof HttpRequest) {
            HttpRequest req = (HttpRequest) msg;

            boolean keepAlive = HttpUtil.isKeepAlive(req);
            FullHttpResponse response = new DefaultFullHttpResponse(req.protocolVersion(), OK,
                                                                    Unpooled.wrappedBuffer(CONTENT));
            response.headers()
                    .set(CONTENT_TYPE, TEXT_PLAIN)
                    .setInt(CONTENT_LENGTH, response.content().readableBytes());

            if (keepAlive) {
                if (!req.protocolVersion().isKeepAliveDefault()) {
                    response.headers().set(CONNECTION, KEEP_ALIVE);
                }
            } else {
                // Tell the client we're going to close the connection.
                response.headers().set(CONNECTION, CLOSE);
            }

            ChannelFuture f = ctx.write(response);

            if (!keepAlive) {
                f.addListener(ChannelFutureListener.CLOSE);
            }
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```
