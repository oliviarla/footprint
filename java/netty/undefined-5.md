# 바이트 버퍼

## 네티의 바이트 버퍼

* 네티에서는 자바의 [ByteBuffer](../variety/nio.md#undefined-1)를 그대로 사용하지 않고 자체적으로 바이트 버퍼를 구현해 사용한다.
* 빈번한 바이트 버퍼 할당 및 해제에 대한 부담을 줄여주어 GC에 대한 부담을 줄여준다.
* 다음은 자바 ByteBuffer와의 차이점이다.
  * 별도 읽기 인덱스, 쓰기 인덱스 보유
  * flip() 메서드 없이 읽기 쓰기 작업 전환
  * 용량을 필요에 따라 확장 가능
  * 바이트 버퍼 풀
  * 내장 복합 버퍼 형식으로 zero copy 달성 가능
  * 자바 바이트 버퍼와 네티 바이트 버퍼 상호 변환
* 읽기, 쓰기에 대해 인덱스가 각각 존재하기 때문에 별도의 flip 메서드 호출 없이 읽기와 쓰기가 가능하다.
* 하나의 바이트 버퍼에 읽기 작업과 쓰기 작업을 동시에 수행할 수도 있다.

## 바이트 버퍼 종류

### 힙 버퍼

* 보조 배열(backing array)이라고 불리며 가장 자주 이용되는 방식이다.
* JVM의 **힙 공간에 데이터를 저장**하여, 풀링이 사용되지 않는 경우 할당과 해제 속도가 빠르다.

```java
ByteBuf heapBuf = ...;
if(heapBuf.hasArray()) {
    byte[] array = heapBuf.array();
    int offset = heapBuf.arrayOffset() + heapBuf.readerIndex(); // 첫번째 바이트
    int length = heapBuf.readableBytes(); // 읽기 가능한 바이트 수
    handleArray(array, offset, length);
}
```

### 다이렉트 버퍼

* JDK의 ByteBuffer 클래스는 네이티브 호출을 통해 운영체제의 커널 영역에 메모리를 할당하도록 허용하여 네이티브 입출력 작업을 호출하기 전후에 버퍼의 내용을 중간 버퍼로 복사하지 않게 할 수 있다.
* 힙 버퍼를 사용하면 소켓 통신 전에 JVM 힙 공간에 저장되어 있는 버퍼를 다이렉트 버퍼로 복사해야 한다. 반면 다이렉트 버퍼는 가비지 컬렉션이 적용되지 않는 **힙 바깥에 저장되어 소켓 통신에 그대로 사용**할 수 있다.
* 버퍼 할당과 해제에 대한 비용이 힙 버퍼 방식에 비해 크다.
* 컨테이너의 데이터를 배열로 접근해야하는 경우 아래와 같이 복사해주는 로직이 필요하므로 힙 버퍼 방식이 편할 수 있다.

```java
ByteBuf directBuf = ...;
if(directBuf.hasArray()) {
    int length = directBuf.readableBytes(); // 읽기 가능한 바이트 수
    byte[] array = new byte[length];
    directBuf.getBytes(directBuf.readableIndex(), array);
    handleArray(array, 0, length);
}

```

### 복합 버퍼

* 여러 ByteBuf의 집합을 사용하는 패턴으로, ByteBuf 인스턴스를 필요에 따라 추가/삭제할 수 있다.
* 네티는 CompositeByteBuf 클래스를 제공하여 복합 버퍼를 사용할 수 있도록 한다.
* 헤더와 본문으로 구성되는 HTTP 메시지를 복합 버퍼로 묶어 사용할 수 있다.
* CompositeByteBuf를 사용하는 소켓 입출력 작업을 최적화해 JDK ByteBuffer에 비해 성능과 메모리 소비 면에서 이점이 있다.

```java
CompositeByteBuf messageBuf = Unpooled.compositeBuffer();
ByteBuf headerBuf = ...;
ByteBuf bodyBuf = ...;
messageBuf.addComponents(headerBuf, bodyBuf);
// ...
messageBuf.removeComponent(0); // 헤더 제거
for (ByteBuf buf : messageBuf) {
    System.out.println(buf.toString());
}

// 다이렉트 버퍼와 비슷한 방식으로 데이터 접근 가능
int length = messageBuf.readableBytes();
byte[] array = new byte[length];
directBuf.getBytes(messageBuf.readableIndex(), array);
handleArray(array, 0, length);
```

## 바이트 버퍼 사용 방법

* ByteBuf 클래스의 사용 방법을 알아본다.

### 버퍼 읽고 쓰기

* 버퍼에 쓸 때는 `writeXXX()` 메서드를 사용하면 된다. writeInt, writeLong, writeChar, writeBytes 등 다양한 메서드를 지원한다.
* 버퍼에서 읽을 때는 `readXXX()` 메서드를 사용하면 된다. 마찬가지로 readInt, readLong, readBytes 등 다양한 메서드를 지원한다.
* 아래 코드에서는 4 bytes 짜리 정수를 바이트 버퍼에 입력하고, 2 bytes만 바이트 버퍼로부터 읽는다.
* 65537은 16진수로 나타내면 0x10001이고, 이 값에 4 bytes 패딩을 하면 0x00010001이 된다. 따라서 앞쪽 2 bytes를 읽으면 1이 된다.

```java
Bytebuf buf = PooledByteBufAllocator.DEFAULT.heapBuffer(11);

buf.writeInt(65537);
assertEquals(4, buf.readableBytes());
assertEquals(7, buf.writableBytes());

assertEquals(1, buf.readShort());
```

* 아래는 읽을 수 있는 바이트를 모두 읽는 코드이다.

```java
ByteBuf buf = ...;
while(buffer.isReadable()) {
    System.out.println(buffer.readByte());
}
```

* getXXX() 메서드와 setXXX() 메서드를 이용해서도 읽기와 쓰기를 할 수 있으나 이 과정에서 **인덱스를 변경시키지 않는다**.
* 아래와 같이 특정 바이트를 수정, 조회하더라도 readerIndex, writerIndex에 변화가 없는 것을 확인할 수 있다.

```java
ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!", Charset.forName("UTF-8"));
System.out.println((char) buf.getByte(0));

int readerIndex = buf.readerIndex();
int writerIndex = buf.writerIndex();

buf.setByte(0, (byte) 'B');
System.out.println((char) buf.getByte(0));

assertEquals(readerIndex, buf.readerIndex());
assertEquals(writerIndex, buf.writerIndex());
```

### 인덱스 관리

* `markReaderIndex/markWriterIndex` 메서드와 `resetReaderIndex/resetWriterIndex` 메서드를 제공하여 인덱스를 지정하거나 초기화할 수 있다.
* `readerIndex(int)`, `writerIndex(int)` 메서드를 통해 지정한 위치로 인덱스를 이동할 수도 있다.

### 특정 바이트 검색

* ByteBufProcessor의 편의성 메서드를 이용해 특정 문자를 검색할 수 있다.
* 다음은 '\r' 문자의 인덱스를 바이트 버퍼에서 찾는 코드이다.

```java
ByteBuf buf = ...;
int index = buffer.forEachByte(ByteBufProcessor.FIND_CR);
```

### 버퍼 초기화

* `clear()` 메서드를 통해 버퍼 데이터를 초기화할 수 있다.

```java
Bytebuf buf = PooledByteBufAllocator.DEFAULT.heapBuffer(11);

buf.writeInt(65537);
assertEquals(4, buf.readableBytes());
assertEquals(7, buf.writableBytes());

buf.clear();
assertEquals(0, buf.readableBytes());
assertEquals(11, buf.writableBytes());
```

* 혹은 `discardReadBytes()` 메서드를 이용해 이미 읽은 부분은 배열에서 제거하고 앞으로 읽을 부분과 기록 가능한 남은 부분만을 배열에서 남길 수 있다.
* 하지만 배열의 앞부분을 제거하고 뒷 부분을 앞으로 땡겨오기 위해 메모리 복사가 이뤄지므로 메모리가 아주 중요한 상황 등 꼭 필요할 경우에만 사용해야 한다.

### 파생 버퍼

* 바이트 버퍼의 일부 혹은 전체를 이용해 새로운 바이트 버퍼 인스턴스를 만들어낼 수 있으며, 내부의 데이터는 공유된 형태이므로 내용을 수정하면 두 인스턴스 수정된 값을 갖게 된다.
* `duplicate()`, `slice()`, `slice(int, int)`, `Unpooled.unmodifiableBuffer(...)`, `order(ByteOrder)`, `readSlice(int)` 메서드를 사용해 파생 버퍼를 만들 수 있다.
* 다음은 slice 메서드를 이용해 buf 객체의 일부만 잘라내어 새로운 파생 버퍼를 만드는 코드이다.

```java
Charset utf8 = Charset.forName("UTF-8");
ByteBuf buf = Unpooled.copiedBuffer("Netty in action!", utf8);
ByteBuf sliced = buf.slice(0, 14);
System.out.println(sliced.toString(utf8));
buf.setByte(0, (byte)'J');
assert buf.getByte(0) == sliced.getByte(0);
```

* 단순히 바이트 버퍼의 복제본이 필요하다면 `copy()`, `copy(int, int)` 메서드를 사용할 수 있으며, 이 경우 내부의 데이터가 공유되지 않는 독립된 객체가 생성된다.

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

## ByteBufHolder

* 실제 데이터 페이로드와 함께 다양한 속성 값을 저장해야 하는 경우 ByteBufHolder를 사용해 페이로드를 저장하는 메시지 객체 구현 시 용이하도록 한다.
* copy(), duplicate() 메서드를 제공하여 ByteBuf의 복제본을 얻을 수 있도록 한다.

## 바이트 버퍼 풀

* 네티는 프레임워크 레벨에서 **바이트 버퍼 풀을 제공**하여 바이트 버퍼를 재사용할 수 있도록 한다.
* 버퍼를 빈번히 할당하고 해제할 때 일어나는 오버헤드와 GC 횟수를 감소시킬 수 있다.
* 버퍼 풀을 사용하는지 아닌지, 그리고 데이터를 힙 메모리에 저장하는지 운영체제의 커널 영역에 저장하는지에 따라 각각 다른 클래스를 제공한다.

<table><thead><tr><th width="144.33333333333331"></th><th>풀링 O</th><th>풀링 X</th></tr></thead><tbody><tr><td>힙 버퍼</td><td>PooledHeapByteBuf</td><td>UnpooledHeapByteBuf</td></tr><tr><td>다이렉트 버퍼</td><td>PooledDirectByteBuf</td><td>UnpooledDirectByteBuf</td></tr></tbody></table>

* 바이트 버퍼 객체를 생성하는 방법은 아래와 같다.

<table><thead><tr><th width="142.33333333333331"></th><th>풀링 O</th><th>풀링 X</th></tr></thead><tbody><tr><td>힙 버퍼</td><td>ByteBufAllocator.DEFAULT.heapbuffer()</td><td>Unpooled.buffer()</td></tr><tr><td>다이렉트 버퍼</td><td>ByteBufAllocator.DEFAULT.directBuffer()</td><td>Unpooled.directBuffer()</td></tr></tbody></table>

* ByteBufAllocator의 참조는 Channel이나 ChannelHandlerContext를 통해 얻을 수 있다.
* ByteBufAllocator에서 제공하는 메서드는 아래와 같다.
  * &#x20;힙 버퍼 혹은 다이렉트 버퍼를 얻을 수 있는 buffer 메서드
  * 힙 버퍼를 얻을 수 있는 heapBuffer 메서드
  * 다이렉트 버퍼를 얻을 수 있는 directBuffer 메서드
  * 복합 버퍼를 얻을 수 있는 compositeBuffer, compositeHeapBuffer, compositeDirectBuffer 메서드
  * 소켓 입출력 작업에 사용될 바이트 버퍼를 얻을 수 있는 ioBuffer 메서드
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

## ByteBufUtil

* ByteBuf를 조작하기 위한 정적 도우미 메서드들을 제공하는 클래스이다.
* ByteBuf 내용을 16진수로 출력하는 hexdump() 메서드는 디버깅을 위한 로깅에 사용될 수 있으며, equals() 메서드도 제공하여 두 ByteBuf 객체가 같은지 비교할 수 있다.

## 참조 카운팅

* 다른 객체에서 더이상 참조하지 않는 객체가 보유한 리소스를 해제해 메모리 사용량과 성능을 최적화하는 기법
* 특정 객체에 대한 활성 참조의 수를 추적하며 참조 카운트가 0이 되면 객체가 할당 해제되어 더이상 사용할 수 없게 만든다.
* 풀링 구현에서 메모리 할당의 오버헤드를 줄이는 데에 반드시 필요하다. 바이트 버퍼의 참조 수를 관리하기 위해 ReferenceCountUtil 클래스의 retain, release 메서드를 사용한다.
