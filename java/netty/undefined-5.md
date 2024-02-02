# 바이트 버퍼

## 네티의 바이트 버퍼

* 네티에서는 자바의 [ByteBuffer](../undefined/nio.md#undefined-1)를 그대로 사용하지 않고 자체적으로 바이트 버퍼를 구현해 사용한다.
* 빈번한 바이트 버퍼 할당 및 해제에 대한 부담을 줄여주어 GC에 대한 부담을 줄여준다.
* 다음은 자바 ByteBuffer와의 차이점이다.
  * 별도 읽기 인덱스, 쓰기 인덱스 보유
  * flip() 메서드 없이 읽기 쓰기 작업 전환
  * 가변 바이트 버퍼
  * 바이트 버퍼 풀
  * 복합 버퍼
  * 자바 바이트 버퍼와 네티 바이트 버퍼 상호 변환
* 읽기, 쓰기 메서드가 실행될 때 각각 해당하는 인덱스를 증가시킨다. 읽기, 쓰기에 대해 인덱스가 각각 존재하기 때문에 별도의 flip 메서드 호출 없이 읽기와 쓰기가 가능하다. 하나의 바이트 버퍼에 읽기 작업과 쓰기 작업을 동시에 수행할 수도 있다.

## 바이트 버퍼 풀

* 네티는 프레임워크 레벨에서 **바이트 버퍼 풀을 제공**하여 바이트 버퍼를 재사용할 수 있도록 한다.
* 버퍼를 빈번히 할당하고 해제할 때 일어나는 GC 횟수를 감소시킬 수 있다.
* 바이트 버퍼를 풀링하기 위해 바이트 버퍼에 참조 수를 기록한다. 이를 관리하기 위해 ReferenceCountUtil 클래스의 retain, release 메서드를 사용한다. (자세한 내용은 나와있지 않다. 🥲)
* 버퍼 풀을 사용하는지 아닌지, 그리고 데이터를 힙 메모리에 저장하는지 운영체제의 커널 영역에 저장하는지에 따라 각각 다른 클래스를 제공한다.

<table><thead><tr><th width="144.33333333333331"></th><th>풀링 O</th><th>풀링 X</th></tr></thead><tbody><tr><td>힙 버퍼</td><td>PooledHeapByteBuf</td><td>UnpooledHeapByteBuf</td></tr><tr><td>다이렉트 버퍼</td><td>PooledDirectByteBuf</td><td>UnpooledDirectByteBuf</td></tr></tbody></table>

* 바이트 버퍼 객체를 생성하는 방법은 아래와 같다.

<table><thead><tr><th width="142.33333333333331"></th><th>풀링 O</th><th>풀링 X</th></tr></thead><tbody><tr><td>힙 버퍼</td><td>ByteBufAllocator.DEFAULT.heapbuffer()</td><td>Unpooled.buffer()</td></tr><tr><td>다이렉트 버퍼</td><td>ByteBufAllocator.DEFAULT.directBuffer()</td><td>Unpooled.directBuffer()</td></tr></tbody></table>

* 네티의 채널에서 사용되는 바이트 버퍼 역시 바이트 버퍼 풀을 이용하며, 이벤트 메서드가 종료되면 바이트 버퍼 객체는 풀로 돌아가게 된다.

```java
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        // ...
        ByteBufAllocator byteBufAllocator = ctx.alloc();
        ByteBuf newBuffer = byteBufAllocator.buffer();
        // ...
        ctx.write(msg);
    }
}
```

## 버퍼 사용

### 버퍼 읽고 쓰기

* 버퍼에 쓸 때는 writeXXX() 메서드를 사용하면 된다. writeInt, writeLong, writeChar, writeBytes 등 다양한 메서드를 지원한다.
* 버퍼에서 읽을 때는 readXXX() 메서드를 사용하면 된다. 마찬가지로 readInt, readLong, readBytes 등 다양한 메서드를 지원한다.
* 아래 코드에서는 4 bytes 짜리 정수를 바이트 버퍼에 입력하고, 2 bytes만 바이트 버퍼로부터 읽는다.
* 65537은 16진수로 나타내면 0x10001이고, 이 값에 4 bytes 패딩을 하면 0x00010001이 된다. 따라서 앞쪽 2 bytes를 읽으면 1이 된다.

```java
Bytebuf buf = PooledByteBufAllocator.DEFAULT.heapBuffer(11);

buf.writeInt(65537);
assertEquals(4, buf.readableBytes());
assertEquals(7, buf.writableBytes());

assertEquals(1, buf.readShort());
```

### 버퍼 초기화

* clear() 메서드를 통해 버퍼 데이터를 초기화할 수 있다.

```java
Bytebuf buf = PooledByteBufAllocator.DEFAULT.heapBuffer(11);

buf.writeInt(65537);
assertEquals(4, buf.readableBytes());
assertEquals(7, buf.writableBytes());

buf.clear();
assertEquals(0, buf.readableBytes());
assertEquals(11, buf.writableBytes());
```

### 가변 크기 버퍼

* 자바의 바이트 버퍼와 달리 네티 바이트 버퍼는 크기가 가변적이다.
* capacity 메서드에 원하는 크기를 입력해 바이트 버퍼의 크기를 변경할 수 있다.
* 기존 존재하는 데이터보다 크기를 작게 설정하면 데이터가 잘리게 된다.

```java
Bytebuf buf = PooledByteBufAllocator.DEFAULT.heapBuffer(11);

buf.writeBytes("hello world");
assertEquals(11, buf.readableBytes());
assertEquals(0, buf.writableBytes());

buf.capacity(5);
assertEquals("hello", buf.toString(Charset.defaultCharset()));
assertEquals(5, buf.capacity());
```

### 부호 없는 값 읽기

* 자바의 모든 원시 데이터형은 모두 부호 있는 데이터로 저장되기 때문에, C 언어로 작성된 애플리케이션과 네트워크 통신 시 부호 없는 데이터를 처리하기 곤란하다.
* 네티에서는 getUnsignedXXX() 메서드를 사용해 부호 없는 값을 읽을 수 있도록 한다.

### 엔디안 변환

* 네티 바이트 버퍼의 기본 엔디안은 자바와 동일하게 빅 엔디안 이다.
* 리틀 엔디안이 필요한 상황에서 order 메서드를 사용해 엔디안을 변환할 수 있다.
* 단, 새로운 바이트 버퍼를 생성하는 것이 아닌 주어진 바이트 버퍼 내용을 공유하는 파생 바이트 버퍼 객체를 만드므로 주의해야 한다.

```java
Bytebuf buf = PooledByteBufAllocator.DEFAULT.heapBuffer(11);
Bytebuf littleEndianBuf = buf.order(ByteOrder.LITTLE_ENDIAN);
```

### 자바 바이트 버퍼 변환

* nioBuffer 메서드를 사용해 자바 바이트 버퍼로 변환할 수 있다.
* 네티 바이트 버퍼의 내부 바이트 배열을 공유하므로 주의해야 한다.

```java
Bytebuf buf = PooledByteBufAllocator.DEFAULT.heapBuffer(11);

ByteBuffer byteBuffer = buf.nioBuffer();
```

* 자바 바이트 버퍼를 네티의 바이트 버퍼로 변환할 수도 있다.

```java
ByteBuffer byteBuffer = ByteBuffer.wrap("hello world".getBytes());

Bytebuf buf = UnPooled.wrappedBuffer(byteBuffer);
```
