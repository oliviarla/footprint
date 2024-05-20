# 웹소켓

## 개념

### 웹소켓의 등장

* 요청-응답 상호작용에 기반을 두는 HTTP 프로토콜과 달리 실시간으로 정보를 갱신해야 하는 상황에 사용되는 프로토콜이다.
* 양방향 트래픽을 위한 단일 TCP 연결을 지원하여 HTTP 풀링 방식보다 효율적이다.

### 프레임

* 웹소켓이 정의하는 특수한 메시지 형식인 프레임이 존재하며, 데이터 프레임과 제어 프레임으로 나뉜다.
* 데이터 프레임은 바이너리 혹은 텍스트 데이터가 존재할 수 있고, 제어 프레임은 close, ping, pong에 대한 데이터가 존재한다.
* 네티는 웹소켓 RFC에서 정의한 6가지 프레임을 위한 구현을 제공한다.
  * BinaryWebSocketFrame
  * TextWebSocketFrame
  * ContinuationWebSocketFrame
  * CloseWebSocketFrame
  * PingWebSocketFrame
  * PongWebSocketFrame
* TextFrameHandler, BinaryFrameHandler, ContinuationFrameHandler 클래스를 직접 정의하여 각 데이터 프레임을 처리할 수 있다.

```java
public static final class TextFrameHandler extends
    SimpleChannelInboundHandler<TextWebSocketFrame> {
    @Override
    public void channelRead0(ChannelHandlerContext ctx,
        TextWebSocketFrame msg) throws Exception {
        // Handle text frame
    }
}

public static final class BinaryFrameHandler extends
    SimpleChannelInboundHandler<BinaryWebSocketFrame> {
    @Override
    public void channelRead0(ChannelHandlerContext ctx,
        BinaryWebSocketFrame msg) throws Exception {
        // Handle binary frame
    }
}

public static final class ContinuationFrameHandler extends
    SimpleChannelInboundHandler<ContinuationWebSocketFrame> {
    @Override
    public void channelRead0(ChannelHandlerContext ctx,
        ContinuationWebSocketFrame msg) throws Exception {
        // Handle continuation frame
    }
}
```

## 네티를 이용해 웹소켓 채팅 구현하기

* 웹소켓을 이용하는 애플리케이션은 HTTP/S 프로토콜로 시작한 후 웹소켓으로 업그레이드하기 위해 업그레이드 핸드쉐이크 과정을 거치게 된다.
* HTTP 엔드포인트가 `/` 인 경우 메인 페이지를 보여주고, `/ws`인 경우 HTTP를 통한 웹소켓 핸드쉐이크로 업그레이드하여 앞으로는 웹소켓을 통해 서비스를 제공하게 된다.

### 웹소켓 서버 부트스트랩

* main() 메서드에서는 프로그램 실행 인자로 입력받은 서버의 포트 번호를 사용해 ChatServer를 구동한다. 프로그램이 종료될 때 ChatServer도 graceful하게 종료되도록 한다.
* channelGroup 필드에는 연결된 모든 웹소켓 채널을 포함하도록 한다.
* 채널 파이프라인 구성을 위해 ChatServerInitializer 클래스를 사용하는데 이는 아래에서 자세히 다룬다.

```java
public class ChatServer {
    private final ChannelGroup channelGroup =
        new DefaultChannelGroup(ImmediateEventExecutor.INSTANCE);
    private final EventLoopGroup group = new NioEventLoopGroup();
    private Channel channel;

    public ChannelFuture start(InetSocketAddress address) {
        ServerBootstrap bootstrap = new ServerBootstrap();
        bootstrap.group(group)
             .channel(NioServerSocketChannel.class)
             .childHandler(createInitializer(channelGroup));
        ChannelFuture future = bootstrap.bind(address);
        future.syncUninterruptibly();
        channel = future.channel();
        return future;
    }

    protected ChannelInitializer<Channel> createInitializer(
        ChannelGroup group) {
        return new ChatServerInitializer(group);
    }

    public void destroy() {
        if (channel != null) {
            channel.close();
        }
        channelGroup.close();
        group.shutdownGracefully();
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.err.println("Please give port as argument");
            System.exit(1);
        }
        int port = Integer.parseInt(args[0]);
        final ChatServer endpoint = new ChatServer();
        ChannelFuture future = endpoint.start(
                new InetSocketAddress(port));
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                endpoint.destroy();
            }
        });
        future.channel().closeFuture().syncUninterruptibly();
    }
}
```

### ChannelPipeline 초기화

* ChatServerInitializer 클래스는 채널 파이프라인을 구성하는 역할을 담당하는 클래스이다.
* 핸들러 별 역할은 다음과 같다.
  * `HttpServerCodec`은 바이트 스트림과 네티의 HttpRequest, HttpContent, LastHttpContent 객체를 인코딩/디코딩하는 역할을 한다.
  * `ChunkedWriteHandler`는 파일의 내용을 기록한다.
  * `HttpObjectAggregator`는 HttpMessage와 HttpContent를 집계하여 FullHttpRequest/FullHttpResponse 객체를 생성한다. 이를 통해 다음 핸들러는 완전한 HTTP 요청만 받게 된다.
  * `HttpRequestHandler`는 직접 아래에서 구현할 클래스로, `/ws` 로 보내지 않은 HTTP 요청을 처리한다.
  * `WebSocketServerProtocolHandler`는 웹소켓 업그레이드 핸드쉐이크를 처리한다.
  * `TextWebSocketFrameHandler`도 직접 아래에서 구현할 클래스로, TextWebSocketFrame과 핸드쉐이크 완료 이벤트를 처리한다.

```java
public class ChatServerInitializer extends ChannelInitializer<Channel> {
    private final ChannelGroup group;

    public ChatServerInitializer(ChannelGroup group) {
        this.group = group;
    }

    @Override
    protected void initChannel(Channel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new HttpServerCodec());
        pipeline.addLast(new ChunkedWriteHandler());
        pipeline.addLast(new HttpObjectAggregator(64 * 1024));
        pipeline.addLast(new HttpRequestHandler("/ws"));
        pipeline.addLast(new WebSocketServerProtocolHandler("/ws"));
        pipeline.addLast(new TextWebSocketFrameHandler(group));
    }
}
```

### HTTP 요청 처리

* 채팅방에 대한 접근을 제공하고, 연결된 클라이언트가 보낸 메시지를 표시한다.
* FullHttpRequest 메시지를 처리하며, keepalive 설정도 처리한다.
* 웹소켓 업그레이드가 요청된 경우 참조 카운트를 retain 메서드를 통해 증가시키고 다음 핸들러인 WebSocketServerProtocolHandler로 넘긴다.
* 이외의 경로로 요청이 들어온 경우 index.html의 내용을 전송해야 한다. 암호화나 압축 요청이 없는 경우 DefaultFileRegion으로 담아 제로 카피 형태로 효율적으로 전송하도록 한다.
* 응답의 끝에는 LastHttpContent를 기록하고 flush하여 FullHttpResponse 형태로 응답을 처리하도록 한다.

```java
public class HttpRequestHandler extends SimpleChannelInboundHandler<FullHttpRequest> {
    private final String wsUri;
    private static final File INDEX;

    static {
        URL location = HttpRequestHandler.class
             .getProtectionDomain()
             .getCodeSource().getLocation();
        try {
            String path = location.toURI() + "index.html";
            path = !path.contains("file:") ? path : path.substring(5);
            INDEX = new File(path);
        } catch (URISyntaxException e) {
            throw new IllegalStateException(
                 "Unable to locate index.html", e);
        }
    }

    public HttpRequestHandler(String wsUri) {
        this.wsUri = wsUri;
    }

    @Override
    public void channelRead0(ChannelHandlerContext ctx,
        FullHttpRequest request) throws Exception {
        if (wsUri.equalsIgnoreCase(request.getUri())) {
            ctx.fireChannelRead(request.retain());
        } else {
            if (HttpHeaders.is100ContinueExpected(request)) {
                send100Continue(ctx);
            }
            RandomAccessFile file = new RandomAccessFile(INDEX, "r");
            HttpResponse response = new DefaultHttpResponse(
                request.getProtocolVersion(), HttpResponseStatus.OK);
            response.headers().set(
                HttpHeaders.Names.CONTENT_TYPE,
                "text/html; charset=UTF-8");
            boolean keepAlive = HttpHeaders.isKeepAlive(request);
            if (keepAlive) {
                response.headers().set(
                    HttpHeaders.Names.CONTENT_LENGTH, file.length());
                response.headers().set( HttpHeaders.Names.CONNECTION,
                    HttpHeaders.Values.KEEP_ALIVE);
            }
            ctx.write(response);
            if (ctx.pipeline().get(SslHandler.class) == null) {
                ctx.write(new DefaultFileRegion(
                    file.getChannel(), 0, file.length()));
            } else {
                ctx.write(new ChunkedNioFile(file.getChannel()));
            }
            ChannelFuture future = ctx.writeAndFlush(
                LastHttpContent.EMPTY_LAST_CONTENT);
            if (!keepAlive) {
                future.addListener(ChannelFutureListener.CLOSE);
            }
        }
    }

    private static void send100Continue(ChannelHandlerContext ctx) {
        FullHttpResponse response = new DefaultFullHttpResponse(
            HttpVersion.HTTP_1_1, HttpResponseStatus.CONTINUE);
        ctx.writeAndFlush(response);
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)
        throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

### 웹소켓 프레임 처리

* TextWebSocketFrame을 처리하는 핸들러로, 채널 그룹 내의 모든 웹소켓 연결을 추적한다.
* userEventTriggered 메서드를 재정의하여 핸드쉐이크가 성공했다는 이벤트를 받으면, 기존에 있던 HttpRequestHandler를 파이프라인에서 제거하고, 모든 웹소켓 클라이언트에 새로운 클라이언트가 연결되었음을 알리고, 새로운 웹소켓 채널을 group에 추가한다.
* TextWebSocketFrame를 수신하면 retain()를 통해 참조 카운트를 증가시키고 group에 전송하여 모든 웹소켓 채널에서 데이터를 받도록 한다.
* 참조 카운트를 증가시키는 이유는 channelRead0 메서드가 반환될 때 참조 카운트가 감소하는데, writeAndFlush 메서드는 비동기이기 때문에 나중에 수행될 때 해제되어버린 데이터에 접근하지 않도록 하기 위함이다.

```java
public class TextWebSocketFrameHandler
    extends SimpleChannelInboundHandler<TextWebSocketFrame> {
    private final ChannelGroup group;

    public TextWebSocketFrameHandler(ChannelGroup group) {
        this.group = group;
    }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx,
        Object evt) throws Exception {
        if (evt == WebSocketServerProtocolHandler
             .ServerHandshakeStateEvent.HANDSHAKE_COMPLETE) {
            ctx.pipeline().remove(HttpRequestHandler.class);
            group.writeAndFlush(new TextWebSocketFrame(
                    "Client " + ctx.channel() + " joined"));
            group.add(ctx.channel());
        } else {
            super.userEventTriggered(ctx, evt);
        }
    }

    @Override
    public void channelRead0(ChannelHandlerContext ctx,
        TextWebSocketFrame msg) throws Exception {
        group.writeAndFlush(msg.retain());
    }
}
```

### SSL 암호화 추가

* 채널파이프라인 초기화 클래스의 생성자에 SslContext를 입력받고 SslHandler 객체를 생성해 채널 파이프라인의 맨 앞에 추가하면 된다.

```java
public class SecureChatServerInitializer extends ChatServerInitializer {
    private final SslContext context;

    public SecureChatServerInitializer(ChannelGroup group,
        SslContext context) {
        super(group);
        this.context = context;
    }

    @Override
    protected void initChannel(Channel ch) throws Exception {
        super.initChannel(ch);
        SSLEngine engine = context.newEngine(ch.alloc());
        engine.setUseClientMode(false);
        ch.pipeline().addFirst(new SslHandler(engine));
    }
}
```

* 아래는 SslContext 객체를 사용해 채널파이프라인 초기화 클래스인 SecureChatServerInitializer의 객체를 생성하여 부트스트랩에서 사용하도록 하는 코드이다.

```java
public class SecureChatServer extends ChatServer {
    private final SslContext context;

    public SecureChatServer(SslContext context) {
        this.context = context;
    }

    @Override
    protected ChannelInitializer<Channel> createInitializer(
        ChannelGroup group) {
        return new SecureChatServerInitializer(group, context);
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.err.println("Please give port as argument");
            System.exit(1);
        }
        int port = Integer.parseInt(args[0]);
        SelfSignedCertificate cert = new SelfSignedCertificate();
        SslContext context = SslContext.newServerContext(
                cert.certificate(), cert.privateKey());
        final SecureChatServer endpoint = new SecureChatServer(context);
        ChannelFuture future = endpoint.start(new InetSocketAddress(port));
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                endpoint.destroy();
            }
        });
        future.channel().closeFuture().syncUninterruptibly();
    }
}
```
