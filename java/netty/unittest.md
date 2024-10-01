# 네티 테스트

## EmbeddedChannel

* 네티에서는 ChannelPipeline에 ChannelHandler 구현을 연결하는 방식을 사용해 복잡한 처리가 필요할 때에도 작고 재사용 가능한 컴포넌트로 분리해 사용할 수 있다.
* 이벤트를 파이프라인을 통해 간단하게 전달할 수 있도록 해준다.
* 인바운드/아웃바운드 데이터를 EmbeddedChannel로 기록하고 ChannelPipeline 끝에 도달하는 항목이 있는지 확인하여 동작을 테스트할 수 있다.
* 인바운드 데이터는 ChannelInboundHandler에서 처리되고, 아웃바운드 데이터는 ChannelOutboundHandler에서 처리된다.
* 제공되는 메서드는 아래와 같다.
  * **writeOutbound** 메서드는 메시지를 채널에 기록하고 아웃바운드 방향으로 ChannelPipeline을 통과시킨다.
  * **readOutbound** 메서드는 처리된 아웃바운드 메시지를 반환한다. 이를 통해 사용자는 예상된 결과가 나왔는지 확인할 수 있다.
  * **writeInbound** 메서드는 메시지를 채널에 기록하고 인바운드 방향으로 ChannelPipeline을 통과시킨다.
  * **readInbound** 메서드는 처리된 인바운드 메시지를 반환한다. 이를 통해 사용자는 예상된 결과가 나왔는지 확인할 수 있다.
  * **finish** 메서드는 EmbeddedChannel을 완료되었다고 표시하고, 전체 인바운드, 아웃바운드 데이터를 읽을 수 있다면 true를 반환하고 채널을 close한다.

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

## 인바운드 테스트

* 아래와 같은 JUnit 테스트에서 EmbeddedChannel을 사용할 수 있다.
* 아래 테스트 코드는 입력된 데이터를 고정된 3바이트 크기의 프레임으로 반환하는 FixedLengthFrameDecoder를 테스트한다.

```java
@Test
public void testFramesDecoded() {
    ByteBuf buf = Unpooled.buffer();
    for (int i = 0; i < 9; i++) {
        buf.writeByte(i);
    }
    ByteBuf input = buf.duplicate();
    EmbeddedChannel channel = new EmbeddedChannel(
        new FixedLengthFrameDecoder(3));
    // write bytes and finish channel
    assertTrue(channel.writeInbound(input.retain()));
    assertTrue(channel.finish());

    // read messages
    ByteBuf read = (ByteBuf) channel.readInbound();
    assertEquals(buf.readSlice(3), read);
    read.release();

    read = (ByteBuf) channel.readInbound();
    assertEquals(buf.readSlice(3), read);
    read.release();

    read = (ByteBuf) channel.readInbound();
    assertEquals(buf.readSlice(3), read);
    read.release();

    assertNull(channel.readInbound());
    buf.release();
}
```

## 아웃바운드 테스트

* 음수를 절댓값으로 변환하는 AbsIntegerEncoder의 테스트코드는 아래와 같다.
* 10개의 음수를 EmbeddedChannel 채널에 write한 후 채널을 완료로 표시한다.
* EmbeddedChannel의 readOutbound 메서드를 통해 음수가 절댓값으로 변환되었는지 확인한다.

```java
@Test
public void testEncoded() {
    ByteBuf buf = Unpooled.buffer();
    for (int i = 1; i < 10; i++) {
        buf.writeInt(i * -1);
    }

    EmbeddedChannel channel = new EmbeddedChannel(
        new AbsIntegerEncoder());
    assertTrue(channel.writeOutbound(buf));
    assertTrue(channel.finish());

    // read bytes
    for (int i = 1; i < 10; i++) {
        assertEquals(i, channel.readOutbound());
    }
    assertNull(channel.readOutbound());
}
```

## 예외 처리 테스트

* 보통 프로그램을 작성할 때 입력이 잘못되거나 데이터가 너무 많을 때 예외를 발생시킨다.
* 채널 파이프라인에서 예외가 발생하면 다른 ChannelHandler에서 exceptionCaught 메서드를 사용해 처리하거나 무시할 수 있다.
* 입력된 프레임의 크기가 최대치를 넘을 경우 바이트를 폐기하고 TooLongFrameException을 발생시키는 FrameChunkDecoder를 테스트하는 코드는 아래와 같다.
* try-catch 문으로 예외가 발생할 것으로 예상되는 구문을 잡아 실패함을 확인할 수 있다.

```java
@Test
public void testFramesDecoded() {
    ByteBuf buf = Unpooled.buffer();
    for (int i = 0; i < 9; i++) {
        buf.writeByte(i);
    }
    ByteBuf input = buf.duplicate();

    EmbeddedChannel channel = new EmbeddedChannel(
        new FrameChunkDecoder(3)); // 3바이트 초과하면 예외 발생

    assertTrue(channel.writeInbound(input.readBytes(2)));
    assertThrows(TooLongFrameException.class, () -> channel.writeInbound(input.readBytes(4)));
    assertTrue(channel.writeInbound(input.readBytes(3)));
    assertTrue(channel.finish());

    // Read frames
    ByteBuf read = (ByteBuf) channel.readInbound();
    assertEquals(buf.readSlice(2), read);
    read.release();

    read = (ByteBuf) channel.readInbound();
    assertEquals(buf.skipBytes(4).readSlice(3), read); // 예외가 발생한 4바이트는 건너뛴다.
    read.release();
    buf.release();
}
```
