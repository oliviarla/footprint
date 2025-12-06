# 7장: 중단 및 종료

## 작업 중단

### 작업 중단

* 사용자가 취소 요청을 하거나, 제한된 시간동안만 작업하기를 원하거나, 오류가 났다거나, 애플리케이션이나 서비스 자체가 종료될 때 실행중인 작업을 취소해야 한다.
* 실행 중인 작업은 취소 요청 플래그를 주기적으로 확인하고 만약 누군가 취소 요청을 했다면 작업을 멈춰야 한다.
* 작업을 쉽게 취소시킬 수 있도록 하려면 작업 취소 요청이 들어왔을 때 어떻게 언제 어떤 일을 해야 하는지 명확히 정의해야 한다.
* 계속해서 대기중인 블로킹 메서드를 중단하려 한다면 인터럽트를 사용할 수 있는데, 작업 취소 기능을 구현하고자 할 때 이러한 인터럽트 방식이 적절하다.

### 인터럽트

* 스레드는 인터럽트 처리 정책을 가져야 한다. 인터럽트 요청이 들어오면 스레드, 서비스 수준의 작업 중단 기능을 제공하고 사용하던 자원을 정리해야 한다.
* 블로킹 메서드를 스스로 실행하기보단 외부 스레드에 위임하는 형태가 많기 때문에, 메서드 실행 중 인터럽트가 발생하면 자신을 호출한 스레드에게 인터럽트 대응을 위임한다.
* 작업 실행 및 취소하는 로직에서는 자기 자신을 수행할 스레드가 어떤 인터럽트 정책을 가지는지 예상할 수 없다.
* `InterruptedException` 이 발생하면 예외를 호출 스택 상위 메서드에 그대로 전달하거나, catch 문 내부에서 `Thread#interrupt` 메서드를 호출해 스레드의 인터럽트 상태를 그대로 유지시켜야 한다.
* 작업 중단 기능을 지원하지 않는 경우 블로킹 메서드를 호출해 인터럽트가 발생하면 자동으로 재시도하도록 구성할 수 있다.
* 작업이 진행되는 과정에서 중간중간 인터럽트 상태를 확인해주면 응답 속도를 높일 수 있다.

> Thread#join 메서드는 스레드가 종료될 때 까지 일정 시간(ms)동안 기다리는데, 만약 일정 시간이 지나도 스레드가 종료되지 않았다면 아무 일도 없이 메서드를 빠져나온다.

### Future를 통한 작업 중단

* `Future#cancel` 메서드를 통해 실행중인 작업을 중단시킬 수 있다.
* `Future#cancel` 메서드에서는 `mayInterruptIfRunning` 이라는 인자를 받는데, 인터럽트에 대응하도록 만들어진 작업이 아니라면 false로 설정해야 한다.
* Executor에서 작업 실행을 위해 생성하는 스레드는 기본적으로 인터럽트가 걸렸을 때 작업을 중단하도록 하는 인터럽트 정책이 있다. 따라서 cancel 메서드의 `mayInterruptIfRunning` 인자를 true로 설정해도 된다.
* 당연하게도 스레드 풀의 스레드에 직접 인터럽트를 걸면 안된다. 작업을 중단하려면 `Future#cancel`을 사용하자.

```java
Future<?> task = taskExec.submit(r);
try {
    task.get(timeout, unit);
} catch (TimeoutException | InterruptedException e) {
    // finally를 통해 작업을 중단시킨다.
} catch (ExecutionException e) {
    // 예외를 다시 던진다.
    throw new RuntimeException(e);
} finally {
    // 작업이 아직 실행중이라면 인터럽트를 건다.
    task.cancel(true);
}
```

* 스레드가 대기 상태에 멈춰있는 경우
  *   java.io 패키지의 동기적 소켓 I/O로 대기중인 경우

      * 연결된 소켓을 직접 닫으면 블로킹되어 있던 read/write 메서드가 중단되고 SocketException을 발생시킨다.

      ```java
      public class ReaderThread extends Thread {
          private final Socket socket;
          private final InputStream in;
          // ...
          
          @Override
          public void interrupt() {
              try {
                  socket.close();
              } catch (IOException ignored) {
              } finally {
                  super.interrupt();
              }
          }
          
          @Override
          public void run() {
              try {
                  byte[] buf = new byte[..];
                  while (true) {
                      int count = in.read(buf);
                      if (count < 0) {
                          break;
                      } else if (count > 0) {
                          processBuffer(buf, count);
                      }
                  }
              } catch (IOException e) {
                  // run 메서드가 끝나면 thread가 종료된다.
              }
          }
      }
      ```
  * java.nio 패키지의 동기적 소켓 I/O로 대기중인 경우
    * InterruptibleChannel에서 대기하고 있는 스레드에 인터럽트를 걸면, ClosedByInterruptException이 발생하면서 채널이 닫힌다. 해당 채널로 작업을 실행하던 스레드에서는 AsyncronousCloseException이 발생한다.
  * Selector를 사용한 비동기적 I/O
    * select 메서드로 대기중일 때 close 메서드를 호출하면, 블로킹이 중단되고 ClosedSelectorException이 발생한다.
  * 락 확보 목적으로 대기
    * lockInterruptibly 메서드를 사용해 락을 확보할 때 까지 대기하면서 인터럽트에 대응할 수 있다.

## 스레드 기반 서비스 중단

* 특정 스레드를 소유한 객체는 해당 스레드를 생성한 객체라고 볼 수 있다.
* 예를 들어 스레드 풀의 스레드들은 스레드 풀 객체가 소유하고 있으며 개별 스레드에 인터럽트를 걸어야 하는 상황이라면 스레드 풀이 책임져야 한다.
* 스레드 기반 서비스가 종료되는 경우 스스로가 소유한 작업 스레드를 종료시켜야 한다. ExecutorService 인터페이스는 이를 구현할 수 있도록 shutdown, shutdownNow 메서드를 제공하고 있다.
  * shutdownNow 메서드를 사용하면 실행중인 모든 스레드의 작업을 중단하고 등록은 됐지만 실행되지 않은 작업 목록을 반환한다. 하지만 실행 시작은 되었지만 완료되지 않은 작업 목록은 알 수 없다.
  * 따라서 직접 작업 수행 중에 ExecutorService가 중단되었고 현재 스레드가 Interrupt된 상태라면 작업 중단 시점의 내용, 추후 재시도할 때 도움이 될만한 내용을 기록하는 로직을 작성해주어야 한다.
* 프로듀서-컨슈머 패턴이 구현되어 있는 프로그램을 종료시키려면 프로듀서와 컨슈머 모두 중단시켜야 한다. 중단시키는 시점에 두 스레드 간에 경쟁 조건이 발생하지 않도록 조심해야 한다.
  * 크기에 제한이 없는 FIFO 큐를 사용하는 경우 poison pill 객체를 추가하면 서비스를 종료하도록 구성할 수 있다. 프로듀서는 해당 객체를 추가하면 다른 객체를 더이상 추가할 수 없도록 구현해야 한다.&#x20;
  * poison pill 객체는 프로듀서 개수와 컨슈머 개수를 정확히 알고 있을 때 사용 가능하다. 프로듀서 개수 만큼의 poison pill 객체가 추가되면 컨슈머를 종료해도 된다.
* 여러 작업을 병렬로 처리해야 하는 상황에서 모든 작업이 종료되는 것을 대기하려면, ExecutorService를 통해 각 작업을 `execute()`하고, 작업 종료를 `shutdown()`으로 대기한다.

## 비정상적인 스레드 종료 처리

* RuntimeException은 보통 try-catch 문으로 잡지 않기 때문에 스레드가 예상치 못하게 종료되는 원인 중 하나이다.
* RuntimeException이 발생하면 현재 시점의 스택 호출 추적 내용을 출력하고 스레드가 종료된다.
* 따라서 내용을 알 수 없는 로직을 수행하기만 하는 스레드의 경우 예외 상황에 대응할 수 있도록 준비하고 종료 시에 외부에 해당 사항을 알려야 한다.
  * 예를 들어 스레드 풀에 속한 스레드가 종료된다면, 이 사실을 스레드 풀에게 알려 새로운 스레드를 생성하도록 해야 한다.
* Thread 클래스의 UncaughtExceptionHandler라는 인터페이스를 구현하여 스레드 객체의 `setUncaughtExceptionHandler` 메서드에 넘겨주면, catch되지 않은 예외로 인해 스레드가 종료될 때 어떤 작업을 할 지 정의할 수 있다. 혹은 ThreadPoolExecutor 생성 시 ThreadFactory 클래스에 넘겨주고, 작업을 execute 메서드로 넘겨 실행해야 한다. 만약 작업을 submit 메서드로 실행하면 Future#get 메서드에서 예외가 ExecutionException에 감싸여 반환된다.

## JVM 종료

### 종료 시점 및 절차

* JVM이 종료되는 경우는 다음과 같다.
  * 일반 스레드가 모두 종료되는 시점
  * System.exit 메서드가 호출된 시점
  * SIGINT 시그널을 받거나 CTRL+C 키를 입력하는 경우 등 여러 상황
* JVM은 종료 시에 Shutdown Hook(종료 훅)에 등록된 훅을 실행시킨다.
  * 여러 훅 간의 실행 순서는 보장되지 않는다.
  * 종료 훅은 thread-safe하며 아무런 가정 없이 올바르게 동작할 수 있도록 방어적으로 구현되어야 한다.
  * 종료 훅을 통해 운영체제에서 자동으로 정리해주지 않는 자원을 정리해야 한다.
* 종료 훅이 모두 완료되면 클래스의 finalize 메서드를 모두 호출하고 종료된다.
* JVM은 종료 시에 애플리케이션 내부의 스레드에 대해 중단 절차를 진행하거나 인터럽트를 걸지 않는다.

### 일반 스레드, 데몬 스레드

* JVM의 스레드는 일반 스레드와 데몬 스레드로 나뉘는데, 데몬 스레드는 main 스레드를 제외한 JVM 내부적으로 사용하기 위한 스레드를 의미한다.&#x20;
* 새로운 스레드가 생성될 때 자신을 생성한 부모 스레드의 데몬 설정 상태를 그대로 사용한다.
  * 따라서 main 스레드에서 생성한 스레드는 모두 기본적으로 일반 스레드이다.
* JVM이 종료될 때 데몬 스레드는 finally 블록의 코드를 실행하지 않고 호출 스택도 원상복구할 수 없도록, 말대로 버려진다.
* 따라서 I/O와 관련된 기능 혹은 자원을 정리하는 기능은 데몬 스레드가 담당하면 안된다. 부수적이고 단순한 작업을 맡기는 정도면 괜찮다.

#### finalize 메서드

* 파일이나 소켓과 같은 일부 자원을 더 이상 사용하지 않을 때 자원을 정리할 수 있도록 하는 메서드이다.
* JVM이 관리하는 스레드에서 직접 호출할 수 있으므로 thread-safe해야 한다.
* [finalize 메서드는 가급적 사용하지 않는 것이 좋다.](../effective-java/2/item8-finalizer-cleaner.md)



