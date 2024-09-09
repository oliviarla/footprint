# 상태 패턴

## 접근

* **발생할 수 있는 상태**들과 **상태를 만드는 행동**들이 있을 때 상태마다 행동이 들어왔을 때의 처리 방법을 정의해두도록 한다.
* 이를 통해 특정 행동이 발생했을 때 if-elseif-else문으로 상태마다 처리방법을 달리 하는 대신 상태 객체의 메서드를 호출하여 작업을 수행하도록 한다.

## 개념

* 어떤 행동이 호출되면 그 행동은 현재 상태를 나타내는 객체에게 위임된다. 객체 내부의 상태가 바뀜에 따라 객체의 행동이 바뀐다.
* 컴포지트를 사용하여 여러 상태 객체를 내부에 두고, 현재 상태 필드를 바꾸어가며 사용한다.
* 마치 객체의 클래스가 바뀌는 것과 같은 효과를 줄 수 있다.

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

* 상태 객체에 행동들이 캡슐화된다. 또한 이 상태들을 관리하는 것은 Context 객체이다. 클라이언트는 Context 객체 내부의 상태를 직접 제어할 필요가 없으며 Context 객체에 원하는 행동만 요청하면 된다.
* 전략 패턴과 동일한 다이어그램 형태이지만 전략 패턴은 클라이언트가 적합한 전략을 가진 객체를 직접 선택해 사용하므로 다르다.

## 장단점

* 장점
  * 각 상태의 행동을 별개 클래스로 분리해 단일 책임 원칙을 지킨다.
  * 각 상태들의 변경에는 닫혀 있고 새로운 상태 클래스 도입에는 열려있어 개방 폐쇄 원칙을 지킨다.
  * 거대한 조건문들을 제거하여 가독성을 높인다.
* 단점
  * 상태 종류가 거의 없거나 상태가 거의 변경되지 않을 때에 적용하면 과하다.

## 사용 방법

* 상태에 의존하는 도메인 클래스를 생성하고, 현재 상태 필드를 둔다.
* 하나의 State 인터페이스를 두고 사용자가 할 수 있는 행동들을 정의한 후, 존재할 수 있는 모든 상태들을 구현한다. 상태 구현체의 내부에서 행동을 구현할 때, 도메인 객체의 현재 상태를 변경할 수 있도록 도메인 객체의 인스턴스를 갖도록 구현해도 된다.
* 도메인 클래스에 각각의 상태 구현체들을 인스턴스 변수로 두고 사용한다.

## 예시

### Redis Lettuce의 RedisStateMachine

* 각종 상태들을 enum 타입으로 두고 상태를 처리하는 메서드를 매핑하여 명령어가 상황에 맞게 처리될 수 있도록 한다.

```java
static class State {

    /**
    * Callback interface to handle a {@link State}.
    */
    @FunctionalInterface
    interface StateHandler {
        
        Result handle(RedisStateMachine rsm, State state, ByteBuf buffer, CommandOutput<?, ?, ?> output,
        Consumer<Exception> errorHandler);
        
    }

    enum Type implements StateHandler {
    
    /**
    * First byte: {@code +}.
    */
    SINGLE('+', RedisStateMachine::handleSingle),
    
    /**
    * First byte: {@code -}.
    */
    ERROR('-', RedisStateMachine::handleError),
    // ...

    }
    // ...
}
    
static State.Result handleSingle(RedisStateMachine rsm, State state, ByteBuf buffer, CommandOutput<?, ?, ?> output,
        Consumer<Exception> errorHandler) {
    ByteBuffer bytes;

    if ((bytes = rsm.readLine(buffer)) == null) {
        return State.Result.BREAK_LOOP;
    }

    if (!QUEUED.equals(bytes)) {
        rsm.safeSetSingle(output, bytes, errorHandler);
    }
    return State.Result.NORMAL_END;
}

// ...
```
