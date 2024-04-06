# 이벤트 루프

## 개념

* 이벤트 루프 구현 방법에는 두 가지 방식이 존재한다.
  * 이벤트 리스너와 이벤트 처리 스레드에 기반한 방법으로, 보통 UI 처리 프레임워크가 사용하는 방법이다. 이벤트를 처리하는 로직을 가진 메서드를 대상 객체의 이벤트 리스너에 등록하는 방식이며, 대부분 이벤트 처리 스레드는 단일 스레드로 구현한다.
  * 이벤트 큐에 이벤트를 등록하고, 이벤트 루프가 이벤트 큐에 접근해 처리하는 방법이다. 이벤트를 처리하기 위한 이벤트 루프 스레드를 여러 개 두면, 가장 먼저 이벤트 큐에 접근한 스레드가 가장 앞에 있는 이벤트를 가져와 수행할 수 있다. 네티는 이t 방식으로 구현된다.
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
* 네티의 이벤트가 발생하는 채널은 하나의 이벤트 루프에만 등록된다.
* 네티 4버전의 모든 입출력 작업과 이벤트는 이벤트 루프에 할당된 스레드에 의해 처리된다.
* 네티 3버전 이하에서는 인바운드 이벤트만 이벤트 루프에서 수행하고 아웃바운드 이벤트는 이벤트 루프 또는 호출하는 스레드에서 수행되었다. 이는 동시에 다양한 스레드에서 접근할 수 있는 문제가 있었다.

### 동작 방식

* 각 이벤트 루프는 이벤트 큐를 가진다. 이벤트 큐는 다른 이벤트 루프로부터 분리되어 공유되지 않기 때문에 각 이벤트 루프는 순차적으로 이벤트를 처리할 수 있다.
* 작업을 호출한 스레드가 이벤트 루프에 속하는 스레드인 경우 작업을 수행하고, 만약 다른 스레드라면 작업을 예약하고 이벤트 큐에 넣는다.
* 이벤트 루프에는 비동기 전송 방식과 동기 전송 방식이 존재한다.
  * 비동기 전송
    * 적은 수의 EventLoop를 EventLoopGroup에 두고 스레드를 매핑하여 다수의 채널을 지원한다.
    * 채널이 새로 생성되면 Round Robin 방식으로 여러 EventLoop 중 하나에 할당된다.
    * EventLoop에서 ThreadLocal을 사용하면 여러 채널에서 비용이 많이 드는 객체나 이벤트를 공유할 수 있다.
  * 동기 전송
    * 채널이 새로 생성되면 하나의 EventLoop를 두고 스레드를 매핑한다.
    * 각 채널의 입출력 이벤트는 EventLoop의 스레드에서 처리된다.

### SingleThreadEventExecutor

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

```java
@Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ChannelFuture channelFuture = ctx.writeAndFlush(msg);
        // 기본 제공 채널 리스너 사용
        channelFuture.addListener(ChannelFutureListener.CLOSE);
        
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
```

## 작업 스케줄링

* 작업을 나중에 실행하거나 주기적으로 실행해야 하는 경우 JDK의 ScheduledExecutorService를 이용할 수 있다.

```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(10);

ScheduledFactory<?> future = executor.schedule(
    new Runnable() {
        @Override
        public void run()
        {
            System.out.println("60 seconds later);
        }
    }, 60, TimeUnit.SECONDS);
)
// ...
executor.shutdown();
```

* 하지만 이 방식은 스레드 풀 형태로 동작하므로 많은 작업을 예약할 경우 스레드가 추가 생성되는 등 병목이 발생할 수 있다.
* 네티는 EventLoop를 이용해 아래와 같이 특정 시점에 작업을 수행하도록 예약할 수 있다.

```java
Channel ch = ...;
ScheduledFuture<?> future = ch.eventLoop().schedule(
    new Runnable() {
        @Override
        public void run()
        {
            System.out.println("60 seconds later");
        }
    }, 60, TimeUnit.SECONDS);
```

* 혹은 특정 시간마다 작업이 수행되도록 할 수도 있다.

```javascript
Channel ch = ...
ScheduledFuture<?> future = ch.eventLoop().scheduleAtFixedRate(
    new Runnable() {
        @Override
        public void run()
        {
            System.out.println("Run every 60 seconds");
        }
    }, 60, 60, TimeUnit.Seconds);
```

* 실행을 취소하거나 상태를 확인하기 위해 ScheduledFuture를 사용할 수 있다. 아래는 실행을 취소시키는 예제이다.

```java
ScheduledFuture<?> future = ch.eventLoop().scheduleAtFixedRate(...);
boolean mayInterruptIfRunning = false;
future.cancel(mayInterruptIfRunngin);
```
