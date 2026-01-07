# ForkJoinPool

## 개념

* ExecutorService의 구현체로, 워커 스레드를 관리하고 스레드 풀 상태나 성능 정보를 제공하기도 한다.

### Work-Stealing Algorithm

* 스레드의 워크로드를 균형있게 유지하기 위해 work-stealing 알고리즘을 사용한다.
* ForkJoinPool은 각 워커 스레드마다 가지는 WorkQueue 객체의 배열(`WorkQueue[]`)을 ForkJoinPool에 저장해둔다.
* `WorkQueue[]`의 홀수 인덱스에는 ForkJoinWorkerThread가 사용하는 스레드 별 큐가 저장되고, 짝수 인덱스는 외부 스레드가 제출한 작업을 저장하는 공유 큐로 사용된다. **즉, 외부 스레드와 내부 스레드의 작업 큐를 구분한다.**
* 만약 deque가 비어있다면, 다른 바쁜 스레드의 deque의 가장 뒤에 있는(tail) 태스크를 가져오거나 global entry queue에서 큰 단위의 태스크를 가져온다.

### commonPool

* 모든 ForkJoinTask를 위한 기본 스레드 풀로, static 블록을 통해 자바 애플리케이션 구동 시 자동으로 생성된다.
* 기본적으로 `Runtime.getRuntime().availableProcessors() - 1` 개수까지 스레드를 생성할 수 있다. 최대 스레드 수는 직접 지정 가능하며, 0 초과 32767 이하여야 한다.
  * 프로세서 수보다 1 적게 설정하는 이유는 시스템의 다른 자원이 사용할 몫을 남겨두기 위함이라고 한다.
* 작업 큐에 작업이 존재하고 작업을 실행할 스레드가 부족할 때, parallelism 값 이하로 스레드가 생성된다.
  *   코드

      ```java
      final void signalWork(WorkQueue[] ws, WorkQueue q) {
      		long c; int sp, i; WorkQueue v; Thread p;
      		while ((c = ctl) < 0L) {                       // too few active
      		    if ((sp = (int)c) == 0) {                  // no idle workers
      		        if ((c & ADD_WORKER) != 0L)            // too few workers
      		            tryAddWorker(c);
      		        break;
      		    }
      		    ...
          }
      }
      private void tryAddWorker(long c) {
          boolean add = false;
          do {
              long nc = ((AC_MASK & (c + AC_UNIT)) |
                         (TC_MASK & (c + TC_UNIT)));
              if (ctl == c) {
                  int rs, stop;                 // check if terminating
                  if ((stop = (rs = lockRunState()) & STOP) == 0)
                      add = U.compareAndSwapLong(this, CTL, c, nc);
                  unlockRunState(rs, rs & ~RSLOCK);
                  if (stop != 0)
                      break;
                  if (add) {
                      createWorker();
                      break;
                  }
              }
          } while (((c = ctl) & ADD_WORKER) != 0L && (int)c == 0);
        }
      ```
* `ForkJoinWorkerThread` 클래스를 통해 스레드가 생성된다.
* 계속해서 다른 스레드의 CPU 바운드 작업을 훔쳐 실행시키면서 idle 스레드가 없도록 사용하는 것이 목적이다.
* 스레드는 작업이 완료될 때 까지 공용 풀에 반환되지 않는다. 따라서 I/O 바운드 작업 시에는 절대 사용하지 말아야 한다.

### 구성 요소

* Task
  * 하나의 작업을 Task 단위로 구분한다.
* Methods
  * `execute(ForkJoinTask<?>)` 메서드를 통해 스레드 풀에 Task를 등록할 수 있다.
  * `invoke(ForkJoinTask<T>)` 메서드를 통해 스레드 풀에 Task를 등록하고 결과가 나올때까지 기다린 후 반환한다.

## Task

### ForkJoinTask

* ForkJoinPool에서 수행되는 가장 기본적인 Task 타입이다.

#### **태스크 시작**

* fork()
  * ForkJoinWorkerThread의 작업 큐 또는 ForkJoinPool의 외부 큐에 Task를 등록할 수 있다.
  * 현재 스레드에서 다른 스레드로 작업을 비동기로 **분할**하여 보낼 수 있다.
* invoke()
  * 현재 스레드에서 바로 작업을 수행하여 결과가 나올 때 까지 기다린 후 결과를 반환한다.
* 태스크를 시작시키고 결과를 얻는 예시는 아래와 같다.

```java
// Case 1: fork() -> join() 호출
ForkJoinTask<?> task = new RecursiveTaskExample();
task.fork();
Result result = task.join();

// Case 2: invoke() 호출
ForkJoinTask<Result> task = new RecursiveTaskExample();
Result result = task.invoke();
```

#### **결과 조회**

* join()
  * 메서드 호출 시 fork한 작업의 완료를 기다린다.
  * 이 때 join()을 호출한 스레드가 대기 상태에 들어가면, ForkJoinPool의 다른 스레드가 유휴 상태에 빠져있지 않도록 Work Stealing Algorithm을 활성화한다.
  * 모든 서브태스크 작업이 완료되면 결과를 합병해 최종 결과를 반환한다.
* get(), get(long, TimeUnit) 메서드로 결과를 조회할 수 있다.

#### **결과 주입**

* complete(V) / completeExceptionally(Throwable) 메서드로 Task를 완료시킬 수 있다.
* 추상 클래스이므로 ForkJoinPool에 태스크를 맡기려면, 직접 구현체를 만들어 입력해야 한다. 다음은 구현해야 하는 추상 메서드의 종류이다.
  * getRawResult
    * join 메서드에서 반환되는 결과를 반환한다. 작업이 완료되지 않았다면 null을 반환한다.
  * setRawResult
    * 외부에서 호출하는 용도가 아니다. 입력받은 인자를 result로 설정한다.
  * exec
    * 외부에서 호출하는 용도가 아니다. Task의 기본 동작을 수행하도록 한다. 정상적으로 완료되었다면 true를 반환하고, 완료가 필요하지 않거나 완료되지 않았다면 false를 반환한다.
* 아래는 CompletableFuture에서 사용되는 Completion이라는 ForkJoinTask이다. CompletableFuture에서는 commonPool의 execute 메서드를 통해 태스크를 실행시킨다.

```java
abstract static class Completion extends ForkJoinTask<Void>
    implements Runnable, AsynchronousCompletionTask {
    volatile Completion next;      // Treiber stack link

    /**
     * Performs completion action if triggered, returning a
     * dependent that may need propagation, if one exists.
     *
     * @param mode SYNC, ASYNC, or NESTED
     */
    abstract CompletableFuture<?> tryFire(int mode);

    /** Returns true if possibly still triggerable. Used by cleanStack. */
    abstract boolean isLive();

    public final void run()                { tryFire(ASYNC); }
    public final boolean exec()            { tryFire(ASYNC); return true; }
    public final Void getRawResult()       { return null; }
    public final void setRawResult(Void v) {}
}
```

### RecursiveTask

* compute 추상 메서드를 구현해야 하는 추상 클래스이다.
* compute 메서드에서는 태스크를 서브 태스크로 분할하는 로직과, 더이상 분할 불가일 때 서브 태스크의 결과를 생산할 알고리즘을 정의한다.

<figure><img src="../../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

```java
forkJoinPool.execute(customRecursiveTask);
int result = customRecursiveTask.join();
```

```java
public class CustomRecursiveTask extends RecursiveTask<Integer> {
    private int[] arr;

    private static final int THRESHOLD = 20;

    public CustomRecursiveTask(int[] arr) {
        this.arr = arr;
    }

    @Override
    protected Integer compute() {
        if (arr.length > THRESHOLD) {
            return ForkJoinTask.invokeAll(createSubtasks()) // 태스크를 분할하여 재귀적으로 호출한다.
              .stream()
              .mapToInt(ForkJoinTask::join) // 각 서브 태스크의 결과를 합친다.
              .sum();
        } else {
            return processing(arr); // 태스크가 충분히 작거나 더이상 분할할 수 없으면 태스크 수행
        }
    }

    private Collection<CustomRecursiveTask> createSubtasks() {
        List<CustomRecursiveTask> dividedTasks = new ArrayList<>();
        dividedTasks.add(new CustomRecursiveTask(
          Arrays.copyOfRange(arr, 0, arr.length / 2)));
        dividedTasks.add(new CustomRecursiveTask(
          Arrays.copyOfRange(arr, arr.length / 2, arr.length)));
        return dividedTasks;
    }

    private Integer processing(int[] arr) {
        return Arrays.stream(arr)
          .filter(a -> a > 10 && a < 27)
          .map(a -> a * 10)
          .sum();
    }
}
```

## ManagedBlocker

* ForkJoinPool은 기본적으로 논블로킹 작업을 처리하도록 설계되어 있다.
* 만약 블로킹 작업을 처리해야 한다면, ForkJoinPool의 managedBlock 메서드의 인자로 입력하여 블로킹될 수 있는 태스크를 전달한다.
* ForkJoinPool의 스레드를 사용하게 되면 블로킹되기 때문에 병렬성을 충분히 보장하지 못하게 된다. 따라서 내부적으로 논블로킹 작업을 처리하는 스레드 개수가 parallelism 이하가 되지 않도록 해야 한다.
* 이를 위해 기존 스레드가 대기중이라면 이를 활성화하여 블로킹 작업을 할당하고, 모두 사용중이라면 새로운 스레드를 생성해 블로킹 작업을 할당한다.
* 이 때 스레드 개수는 최대 **32767까지만 생성 가능**하다.
  * ```java
    private boolean tryCompensate(WorkQueue w) {
            boolean canBlock;
            WorkQueue[] ws; long c; int m, pc, sp;
            if (w == null || w.qlock < 0 ||           // caller terminating
                (ws = workQueues) == null || (m = ws.length - 1) <= 0 ||
                (pc = config & SMASK) == 0)           // parallelism disabled
                canBlock = false;
            else if ((sp = (int)(c = ctl)) != 0)      // release idle worker
                canBlock = tryRelease(c, ws[sp & m], 0L);
            else {
                int ac = (int)(c >> AC_SHIFT) + pc;
                int tc = (short)(c >> TC_SHIFT) + pc;
                int nbusy = 0;                        // validate saturation
                for (int i = 0; i <= m; ++i) {        // two passes of odd indices
                    WorkQueue v;
                    if ((v = ws[((i << 1) | 1) & m]) != null) {
                        if ((v.scanState & SCANNING) != 0)
                            break;
                        ++nbusy;
                    }
                }
                if (nbusy != (tc << 1) || ctl != c)
                    canBlock = false;                 // unstable or stale
                else if (tc >= pc && ac > 1 && w.isEmpty()) {
                    long nc = ((AC_MASK & (c - AC_UNIT)) |
                               (~AC_MASK & c));       // uncompensated
                    canBlock = U.compareAndSwapLong(this, CTL, c, nc);
                }
                **else if (tc >= MAX_CAP ||
                         (this == common && tc >= pc + commonMaxSpares))
                    throw new RejectedExecutionException(
                        "Thread limit exceeded replacing blocked worker");**
                else {                                // similar to tryAddWorker
                    boolean add = false; int rs;      // CAS within lock
                    long nc = ((AC_MASK & c) |
                               (TC_MASK & (c + TC_UNIT)));
                    if (((rs = lockRunState()) & STOP) == 0)
                        add = U.compareAndSwapLong(this, CTL, c, nc);
                    unlockRunState(rs, rs & ~RSLOCK);
                    canBlock = add && createWorker(); // throws on exception
                }
            }
            return canBlock;
        }
    ```
* CompletableFuture와의 관계
  * CompletableFuture의 get(), join() 메서드 호출 시 이 ManagedBlocker를 호출하여 블로킹 작업을 위임한다.
  * ForkJoinPool에 속한 스레드가 호출하지 않았다면, 기본적으로 해당 스레드가 대기하게 된다.
  * 만약 get(), join() 메서드가 ForkJoinPool에 속한 스레드에서 호출되었다면 병렬성을 위해 앞서 소개한 방식대로 새로운 스레드를 만들거나 유휴 상태의 스레드를 통해 대기하게 된다.
* 본 인터페이스에 속한 메서드의 의미와 역할은 다음과 같다.
  * isReleasable()
    * 블로킹이 더 이상 필요 없다면 true를 반환한다.
    * 블로킹 조건이 충족되었는지 확인하는 용도로 사용한다.
  * block()
    * 현재 스레드를 블로킹하고 작업을 수행한다.
    * 예를 들어 락이나 특정 조건, 외부 응답 등을 기다리는 블로킹 작업을 구현한다.

## 병렬 스트림과 통합

* 병렬 스트림을 사용하는 경우 ForkJoinPool의 commonPool을 사용하게 된다.
* 스트림을 재귀적으로 분할하고, 각 서브스트림을 서로 다른 스레드에 할당하고, 결과를 하나의 값으로 합치는 오버헤드가 발생하기 때문에, 코어 간 데이터 전송 시간보다 훨씬 오래걸리는 작업만 병렬화하는것이 바람직하다.
* 소스 데이터가 크지 않거나, 소스 데이터의 자료구조를 분할하는 데에 오버헤드가 큰 경우 사용하면 안된다.
  *   예를 들어 LinkedList를 소스로 사용하는 경우 모든 요소를 탐색해야 분할이 가능하므로 적절하지 않다.

      ```java
      list.stream().parallel().forEach(System.out::println);
      ```
  * IntStream#iterate 과 같이 정확히 정해진 범위가 없는 경우 사용하면 안된다.

### 병렬 스트림 동작 방식

* 스트림을 분리할 때에는 Spliterator를 사용한다. 앞서 살펴본 RecursiveTask가 하나의 Task를 여러 Task로 분리했던 방식과 유사하게 하나의 Spliterator를 여러 Spliterator로 분리한다.
* 이렇게 분리된 각 태스크는 ForkJoinPool에 의해 병렬적으로 작업을 수행하게 된다.
* 아래와 같이 단순한 병렬 스트림을 사용 시,Streams$RangeIntSpliterator에 의해 적절한 단위로 쪼개어 작업이 수행된다.

```java
IntStream.range(1, 100_000_000)
  .parallel()
  .forEach(i -> System.out.println(i * 20));
```

```java
private static <S, T> void doCompute(ForEachOrderedTask<S, T> task) {
        Spliterator<S> rightSplit = task.spliterator, leftSplit;
        long sizeThreshold = task.targetSize;
        boolean forkRight = false;
        while (rightSplit.estimateSize() > sizeThreshold &&
               (leftSplit = rightSplit.trySplit()) != null) {
            **ForEachOrderedTask<S, T> leftChild =
                new ForEachOrderedTask<>(task, leftSplit, task.leftPredecessor);
            ForEachOrderedTask<S, T> rightChild =
                new ForEachOrderedTask<>(task, rightSplit, leftChild);**
        ...
```

**출처**

* [https://www.baeldung.com/java-fork-join](https://www.baeldung.com/java-fork-join)
* Java adopt-1.8
