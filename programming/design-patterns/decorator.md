# 데코레이터 패턴

## 접근

* 컴포지션을 통해 객체의 행동을 확장한다면, 런타임에 동적으로 행동을 설정할 수도 있고 객체에 여러 행동을 추가해도 기존 코드에 영향이 최소한으로 가게 된다.
* 클래스는 확장에는 열려있어야 하지만 변경에는 닫혀있어야 한다는 객체 지향의 OCP원칙에 따라 설계해야 한다. 즉 요구 사항에 의해 새로운 기능이 추가되거나 기존 기능이 변경되더라도, 기존 코드에 변경 없이 새로운 코드를 추가할 수 있어야 한다는 의미이다.

## 개념

* 객체에 추가 요소를 동적으로 더할 수 있도록 데코레이터를 사용하는 디자인 패턴이다.
* 데코레이터를 사용하면 서브 클래스를 만드는 것 보다 훨씬 유연하게 기능을 확장할 수 있다.
* 예를 들어 여러 기능들을 조합해야 하는 경우 서브 클래스를 만들다보면 클래스 폭발 현상이 발생할 수 있다. 데코레이터 패턴은 이러한 기능 조합을 원활하게 만들어준다.
* 상속과 컴포지션을 통해 Decorator 추상 클래스는 Component 인터페이스를 구현하는 동시에 필드로 가지며, 필수 메서드와 부가 기능을 구현한다.
* 상속하는 이유는 Component와 타입을 동일하게 맞추기 위함이고, 필드로 갖는 것은 행동을 상속받기 위함이다.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## 장단점

* 장점
  * 객체에 부가 요소를 동적으로 추가할 수 있어 여러 행동을 합성할 수 있다.
  * 여러 행동을 작은 클래스로 나누어 표현하므로 단일 책임 원칙을 지킨다.
* 단점
  * 특정 타입에 의존하는 클라이언트 코드에 데코레이터를 적용하면 제대로 활용할 수 없으므로 아무데나 적용하면 안된다.
  * 많은 클래스가 추가되어야 하므로 구조가 복잡해진다.
  * 클라이언트가 구성 요소를 초기화할 때 코드가 복잡해질 수 있.

## 사용 방법

* 인터페이스 혹은 추상 클래스를 정의하여 데코레이터의 기반이 되는 구성 요소를 만든다.
* 이를 구현하거나 상속받은 클래스들은 이제 데코레이터 클래스의 대상이 될 수 있다.
* 원하는 부가 기능들을 분리하여 각각의 데코레이터 클래스로 만든다. 이 때 앞서 정의한 인터페이스 혹은 추상 클래스를 구현/상속하는 동시에 필드로 가지도록 한다.

## 예시

### &#x20;Java I/O

* I/O에는 다양한 방식이 존재하며, 이를 지원하기 위해 자바에서는 데코레이터 패턴을 사용한다.
* InputStream은 추상 데코레이터 클래스 역할을 하며, 데코레이터에 감싸이는 컴포넌트 역할을 하는 클래스들이 존재한다. 그리고 데코레이터 클래스들이 존재한다. 이 때 데코레이터에서는 입력받은 데이터를 원하는 형태로 가공하도록 한다.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### Lettuce의 ChannelWriter

* Lettuce는 Netty를 기반으로 하는 라이브러리이다. 따라서 Netty에서 제공해주는 Channel에 write하는 역할을 담당하는 클래스가 필요하다.
* 가장 기본적으로 Channel에 데이터를 write하는 클래스는 DefaultEndpoint이다.&#x20;

```java
public class DefaultEndpoint implements RedisChannelWriter, Endpoint, PushHandler {
    // ...
    protected volatile Channel channel; // Netty의 Channel
    // ...
    private void writeToChannelAndFlush(Collection<? extends RedisCommand<?, ?, ?>> commands) {
        // ...
        if (reliability == Reliability.AT_MOST_ONCE) {

            // cancel on exceptions and remove from queue, because there is no housekeeping
            for (RedisCommand<?, ?, ?> command : commands) {
                channelWrite(command).addListener(AtMostOnceWriteListener.newInstance(this, command));
            }
        }

        if (reliability == Reliability.AT_LEAST_ONCE) {

            // commands are ok to stay within the queue, reconnect will retrigger them
            for (RedisCommand<?, ?, ?> command : commands) {
                channelWrite(command).addListener(RetryListener.newInstance(this, command));
            }
        }

        channelFlush();
    }

    private void channelFlush() {

        if (debugEnabled) {
            logger.debug("{} write() channelFlush", logPrefix());
        }

        channel.flush();
    }

    private ChannelFuture channelWrite(RedisCommand<?, ?, ?> command) {

        if (debugEnabled) {
            logger.debug("{} write() channelWrite command {}", logPrefix(), command);
        }

        return channel.write(command);
    }
    // ...
}
```

* Lettuce는 Redis 클러스터에 요청을 보내는 기능도 지원한다. 이 때 어떤 클러스터의 노드에 요청을 보낼 지 정하기 위한 ClusterDistributionChannelWriter 클래스를 제공하는데, 이는 데코레이터 클래스와 유사하다.
* 기본적인 요청을 write하기 위한 RedisChannelWriter 필드를 가지고 있으며, RedisChannelWriter라는 타입에 할당될 수 있도록 인터페이스를 구현하고 있다.

> Redis의 해시 슬롯에 따라 알맞는 노드에 연산을 보내는 부분은 데코레이터 패턴의 쓰임과 달라 설명하지 않았다.

```java
class ClusterDistributionChannelWriter implements RedisChannelWriter {

    private final RedisChannelWriter defaultWriter;
    // ...
}
```

* RedisChannelHandler는 클러스터이든 Standalone이든 상관 없이 사용되는 클래스이다. 만약 클러스터 환경인 경우에는 channelWriter 필드에 `ClusterDistributionChannelWriter`를 저장해둔다. Standalone이라면 channelWriter 필드에 `DefaultEndpoint`를 저장해둔다.

```java
public abstract class RedisChannelHandler<K, V> implements Closeable, ConnectionFacade {
    private final RedisChannelWriter channelWriter;
    
    protected <T> RedisCommand<K, V, T> dispatch(RedisCommand<K, V, T> cmd) {

        if (debugEnabled) {
            logger.debug("dispatching command {}", cmd);
        }

        if (tracingEnabled) {

            RedisCommand<K, V, T> commandToSend = cmd;
            TraceContextProvider provider = CommandWrapper.unwrap(cmd, TraceContextProvider.class);

            if (provider == null) {
                commandToSend = new TracedCommand<>(cmd,
                        clientResources.tracing().initialTraceContextProvider().getTraceContext());
            }

            return channelWriter.write(commandToSend);
        }

        return channelWriter.write(cmd);
    }
}
```
