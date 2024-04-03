---
description: 이벤트 핸들러를 상속받아 구현한 코덱에 대해 알아본다.
---

# 코덱

## 개념

* 네티의 ChannelInboundHandler와 ChannelOutboundHandler는 각각 인코더와 디코더 역할을 한다.
* 데이터 전송 시에는 인코더를 사용해 패킷으로 변환하고, 수신 시에는 디코더를 사용해 패킷을 데이터로 변환한다.
* 인코더 클래스의 경우 ChannelOutboundHandlerAdapter를 상속받은 추상 클래스인 MessageToMessageEncoder를 상속받아 encode 메서드를 구현하는 템플릿 메서드 패턴으로 구현된다. 이를 통해 다양한 종류의 인코더/디코더를 제공할 수 있다.
* 아래와 같이 write 이벤트가 발생할 경우 ChannelOutboundHandler에서 데이터를 인코딩해주게 된다.

<figure><img src="../../.gitbook/assets/image (5).png" alt="" width="375"><figcaption></figcaption></figure>

## 종류

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
