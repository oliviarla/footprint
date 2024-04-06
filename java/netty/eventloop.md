# 이벤트 루프

## 개념

* 이벤트 루프 구현 방법에는 두 가지 방식이 존재한다. 네티는 후자의 방식으로 구현된다.
  * 이벤트 리스너와 이벤트 처리 스레드에 기반한 방법으로, 보통 UI 처리 프레임워크가 사용하는 방법이다. 이벤트를 처리하는 로직을 가진 메서드를 대상 객체의 이벤트 리스너에 등록하는 방식이며, 대부분 이벤트 처리 스레드는 단일 스레드로 구현한다.
  * 이벤트 큐에 이벤트를 등록하고, 이벤트 루프가 이벤트 큐에 접근해 처리하는 방법이다. 이벤트를 처리하기 위한 이벤트 루프 스레드를 여러 개 두면, 가장 먼저 이벤트 큐에 접근한 스레드가 가장 앞에 있는 이벤트를 가져와 수행할 수 있다.
* 이벤트 루프
  * 이벤트를 실행하기 위한 무한루프 스레드
  * 객체에서 이벤트가 발생하면 이벤트 큐에 추가하고, 이벤트 루프는 이벤트 큐에 존재하는 이벤트를 꺼내 수행한다.
  * 이벤트 루프가 처리한 이벤트의 결과를 콜백 패턴 혹은 퓨처 패턴으로 반환할 수 있으며, 네티는 두 방식 모두 지원한다.

## 단일 스레드 및 다중 스레드 이벤트 루프

* 이벤트를 처리하는 스레드가 하나인지, 두개 이상인지에 따라 나뉜다.
* 단일 스레드 이벤트 루프
  * 구현이 단순하고 예측 가능한 동작을 보장한다.
  * 이벤트가 발생한 순서대로 처리할 수 있다.
  * 다중 코어 CPU를 효율적으로 사용하지 못해 처리 시간이 오래걸리는 이벤트가 존재하면 나중에 들어온 이벤트의 수행 시간도 점점 뒤로 밀려나게 된다.
* 다중 스레드 이벤트 루프
  * 단일 스레드 이벤트 루프에 비해 구현이 복잡하지만 다중 코어 CPU를 효율적으로 사용한다.
  *   여러 이벤트 루프 스레드가 이벤트 큐에 접근하여 스레드 경합이 발생하고, 이벤트의 발생 순서와 실행 순서가 일치하지 않는다.

      > 스레드 경합
      >
      > 다중 스레드 애플리케이션에서 각 스레드가 공유 자원의 단일 액세스 권한(락)을 획득하기 위해 경합을 벌이는 것으로, 스레드가 많아질수록 CPU 자원을 많이 소비하는 작업이다.
  * 스레드 개수를 너무 많이 설정하거나 제한하지 않으면 과도한 GC가 발생하거나 OOM 에러가 발생할 수 있다.

## 네티의 이벤트 루프

* 네티는 단일 스레드 이벤트 루프와 다중 스레드 이벤트 루프를 모두 사용할 수 있다.
* 네티는 **다중 스레드 이벤트 루프를 사용하더라도 이벤트의 발생 순서와 실행 순서를 보장**한다.
* 네티의 이벤트가 발생하는 채널은 하나의 이벤트 루프에만 등록된다.&#x20;
* 각 이벤트 루프는 이벤트 큐를 가진다. 즉, 이벤트 큐를 공유하지 않기 때문에 각 이벤트 루프는 순차적으로 이벤트를 처리할 수 있게 된다.
* 네티는 이벤트 처리를 위해 SingleThreadEventExecutor를 사용한다.
* 아래는 네티 4.1 버전의 SingleThreadEventExecutor다.&#x20;
* taskQueue를 두어 이벤트를 Runnable 타입으로 저장하고, pollTaskFrom을 통해 이벤트를 하나 가져온다.
* 이벤트 큐에 입력된 모든 task를 수행하기 위해 runAllTasks() 메서드를 사용할 수 있다. 내부적으로 taskQueue에서 이벤트를 하나씩 가져와 run() 메서드를 통해 수행한다.

```java
public abstract class SingleThreadEventExecutor extends AbstractScheduledEventExecutor implements OrderedEventExecutor {
    private final Queue<Runnable> taskQueue;
    
    protected Runnable pollTask() {
        assert inEventLoop();
        return pollTaskFrom(taskQueue);
    }

    protected static Runnable pollTaskFrom(Queue<Runnable> taskQueue) {
        for (;;) {
            Runnable task = taskQueue.poll();
            if (task != WAKEUP_TASK) {
                return task;
            }
        }
    }
    
    protected boolean runAllTasks() {
        assert inEventLoop();
        boolean fetchedAll;
        boolean ranAtLeastOne = false;

        do {
            fetchedAll = fetchFromScheduledTaskQueue();
            if (runAllTasksFrom(taskQueue)) {
                ranAtLeastOne = true;
            }
        } while (!fetchedAll); // keep on processing until we fetched all scheduled tasks.

        if (ranAtLeastOne) {
            lastExecutionTime = getCurrentTimeNanos();
        }
        afterRunningAllTasks();
        return ranAtLeastOne;
    }

    protected final boolean runAllTasksFrom(Queue<Runnable> taskQueue) {
        Runnable task = pollTaskFrom(taskQueue);
        if (task == null) {
            return false;
        }
        for (;;) {
            safeExecute(task); // 내부적으로 Runnable.run()이 실행됨
            task = pollTaskFrom(taskQueue);
            if (task == null) {
                return true;
            }
        }
    }
}
```

## 네티의 비동기 I/O 처리

* 네티의 비동기 I/O 메서드 호출의 결과를 ChannelFuture 객체로 돌려받을 수 있다.
* ChannelFuture 객체에 채널 리스너를 등록해두면, 비동기 작업이 완료되었을 때 특정 동작을 수행하도록 할 수 있다.
* 네티에서 기본 제공하는 채널 리스너는 아래와 같다.
  * ChannelFutureListener.CLOSE: 작업 완료 이벤트를 수신하면 무조건 ChannelFuture에 포함된 채널을 닫는다.
  * ChannelFutureListener.CLOSE\_ON\_FAILURE : 작업 완료 이벤트를 수신했는데 결과가 실패일 때 채널을 닫는다.
  * ChannelFutureListener.FIRE\_EXCEPTION\_ON\_FAILURE : 작업 완료 이벤트를 수신했는데 결과가 실패일 때 채널 예외 이벤트를 발생시킨다.
* 아래는 네티의 채널 리스너를 등록하여 데이터 전송이 완료되면 소켓 채널을 닫도록 구현한 코드이다. 커스텀 채널 리스너를 구현해 사용할 수도 있다.

<pre class="language-java"><code class="lang-java">@Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ChannelFuture channelFuture = ctx.writeAndFlush(msg);
        // 기본 제공 채널 리스너 사용
<strong>        channelFuture.addListener(ChannelFutureListener.CLOSE);
</strong>        
        // 커스텀 채널 리스너 사용
        channelFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                System.out.println("전송한 Byte : " + writeMessageSize);
                future.channel().close();
            }
        });
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
</code></pre>
