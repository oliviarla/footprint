# CompletableFuture

## 개념

* CompletionStage, Future 인터페이스를 구현하는 클래스이다.
  * Future: `complete` 되었을 때 value, status 값을 조회할 수 있다. 작업을 cancel 시킬 수도 있다.
  * CompletionStage: `complete` 되었을 때 등록된 함수 및 작업을 수행하도록 할 수 있다.
* complete, completeExceptionally, cancel 메서드를 통해 complete 된다. 이 메서드들이 여러 스레드에 의해 동시에 호출된다 하더라도 단 하나의 메서드만 성공하게 된다.

## API 및 내부 동작

* 내부적으로 결과를 저장하는 result 변수를 가지고 있으며, `complete` 시에 수행할 작업들을 stack이라는 Completion 타입의 변수에 LinkedList와 유사하게 체이닝하여 저장해둔다.
* 코드
  * ```java
    final boolean tryPushStack(Completion c) {
        Completion h = stack; // 기존에 있던 stack 변수 소환
        lazySetNext(c, h); // 새로운 Completion 객체의 next 필드를 기존 stack 변수로 설정
        return UNSAFE.compareAndSwapObject(this, STACK, h, c); // stack 변수를 새로운 Completion 객체로 대체
    }

    static void lazySetNext(Completion c, Completion next) {
        UNSAFE.putOrderedObject(c, NEXT, next);
    }

    final void cleanStack() {
        for (Completion p = null, q = stack; q != null;) {
            Completion s = q.next;
            if (q.isLive()) {
                p = q;
                q = s;
            }
            else if (p == null) {
                casStack(q, s);
                q = stack;
            }
            else {
                p.next = s;
                if (p.isLive())
                    q = s;
                else {
                    p = null;  // restart
                    q = stack;
                }
            }
        }
    }
    ```

{% hint style="success" %}
**공통 동작**

* async가 붙은 메서드 중 executor 인자를 주지 않았다면 기본적으로 ForkJoinPool의commonPool에서 작업이 수행된다. executor 인자를 주었다면 executor를 통해 작업이 수행된다.
* async가 메서드에 붙어있지 않다면 기본적으로 complete 메서드를 수행한 스레드에서 계속해서 작업이 수행된다.&#x20;
{% endhint %}

#### 생성

* **기본 생성자**
  * 아무 인자도 받지 않는 기본 생성자를 사용해 CompletableFuture 객체를 생성할 수 있다.
* **completedFuture(U value)**
  * 이미 `complete`된 상태이고 결과가 존재하는 CompletableFuture 객체를 생성하는 static API이다.
* **supplyAsync**
  *   Supplier를 인자로 받아 별도 스레드풀에서 실행하도록 하는 CompletableFuture를 반환한다.

      > Supplier: `() → T`
* **runAsync**
  * Runnable를 인자로 받아 별도 스레드풀에서 실행하도록 하는 CompletableFuture를 반환한다.
  *   작업이 완료되면 null을 반환한다.

      > Runnable: `() → void`
* **allOf / anyOf**
  * 여러 CompletableFuture 객체들을 트리 구조로 결합하여, 모든 작업이 완료되거나(allOf) 하나의 작업이라도 완료(anyOf)될 때 `complete` 되는 새로운 CompletableFuture 객체를 반환한다.
  * 하나의 CompletableFuture에서 예외가 발생했다면, CompletionException이 반환된다.

#### 결과 조회

* **get() / get(long timeout, TimeUnit unit)**
  * complete될 때 까지 기다린다.
  * 예외는 ExecutionException으로 감싸 던진다.
* **join()**
  * get() 과 달리 checked exception을 unchecked exception인 CompletionException으로 감싸 던진다.
  * 대기중일 때 스레드를 interrupt시킬 수 없다.
* **getNow(T valueIfAbsent)**
  * 현재 complete되어 있는 상태라면 값을 반환하고, 아니라면 입력된 인자를 반환한다.
* **isDone()**
  * 예외 발생, cancel에 관계 없이 result 내부 변수가 null이 아니라면 성공한다.
* **isCancelled()**
  * 현재 result 내부 변수에 CancellationException 객체가 AltResult로 감싸 저장되어 있다면 true를 반환한다.
* **isCompletedExceptionally()**
  * 현재 result 내부 변수에 Exception 객체가 AltResult로 감싸 저장되어 있다면 true를 반환한다.

#### 결과 등록

* **complete(T value)**
  * value 인자를 result 내부 변수에 할당한다.
  * postComplete 내부 메서드를 통해 등록된 콜백이 수행된다.
* **completeExceptionally(Throwable ex)**
  * ex 인자를 AltResult 객체로 감싼 후, result 내부 변수에 할당한다.
  * postComplete 내부 메서드를 통해 등록된 콜백이 수행된다.
* **cancel(boolean mayInterruptIfRunning)**
  * CancellationException 객체를 AltResult 객체로 감싼 후, result 내부 변수에 할당한다.
  * postComplete 내부 메서드를 통해 등록된 콜백이 수행된다.
  * 작업을 어느 스레드에서 비동기적으로 수행하고 있는지에 대한 정보를 관리하지 않으므로, mayInterruptIfRunning 값을 true로 주더라도 기존 작업이 interrupt할 수 없다.

***

* **obtrudeValue / obtrudeException**
  * 이미 `complete`된 상태이더라도 결과를 덮어쓴다.
  * postComplete 내부 메서드를 통해 등록된 콜백이 수행된다.

#### 콜백 등록

* **thenApply**
  * Function 객체를 인자로 받아, `complete` 된 후에 결과를 기반으로 특정 작업을 수행하여 새로운 결과를 반환한다.
  *   반환 타입이 CompletableFuture\<R>으로 바뀐다.

      > Function: `T → R`
* **thenAccept**
  * Consumer 객체를 인자로 받아, `complete` 된 후에 결과를 기반으로 특정 작업을 수행한다.
  *   반환 타입이 CompletableFuture\<Void>으로 바뀐다.

      > Consumer: `T → void`
* **thenRun**
  * Runnable 객체를 인자로 받아, `complete` 된 후에 특정 작업을 수행한다.
  *   반환 타입이 CompletableFuture\<Void>으로 바뀐다.

      > Runnable: `() → void`
* **whenComplete**
  * BiConsumer 객체를 인자로 받아, `complete` 된 후에 (결과, 예외)를 기반으로 특정 작업을 수행한다.
  *   반환 타입이 CompletableFuture\<T>로 그대로이다.

      > BiConsumer: `(T, E) → void`
* **handle**
  * BiFunction 객체를 인자로 받아, `complete` 된 후에 (결과, 예외)를 기반으로 특정 작업을 수행하여 새로운 결과를 반환한다.
  *   반환 타입이 CompletableFuture\<R>로 바뀐다.

      > BiFunction: `(T, E) → R`
* **thenCompose**
  * Function 객체를 인자로 받아, `complete` 된 후에 CompletableFuture을 반환하는 또다른 작업을 수행한다.
  *   반환 타입이 CompletableFuture\<U>로 바뀐다.

      > Function: `T → ? extends CompletionStage<U>`
* **thenCombine**
  * CompletionStage, BiFunction 객체를 인자로 받는다.
  * BiFunction은 CompletableFuture\<T>, CompletionStage\<U>의 결과를 입력받아 하나의 값을 만들어 반환한다.
  *   반환 타입이 CompletableFuture\<V>로 바뀐다.

      > BiFunction: `(T, U) → V`
* **thenAcceptBoth**
  * CompletionStage, BiConsumer 객체를 인자로 받는다.
  * BiConsumer는 CompletableFuture\<T>, CompletionStage\<U>의 결과를 입력받아 특정 작업을 수행한다.
  *   반환 타입이 CompletableFuture\<Void>로 바뀐다.

      > BiConsumer: `(T, U) → void`
* **runAfterBoth**
  * CompletionStage, Runnable 객체를 인자로 받는다.
  * CompletableFuture\<T>, CompletionStage\<?>가 모두 `complete` 된 후에 특정 작업을 수행한다.
* **applyToEither**
  * CompletionStage, Function 객체를 인자로 받는다.
  * CompletableFuture\<T>, CompletionStage\<? extends T> 중 먼저 `complete`된 것의 결과를 입력받아 하나의 값을 만들어 반환한다.
  *   반환 타입이 CompletableFuture\<U>로 바뀐다.

      > Function: `T → U`
* **acceptEither**
  * CompletionStage, Consumer 객체를 인자로 받는다.
  * CompletableFuture\<T>, CompletionStage\<? extends T> 중 먼저 `complete`된 것의 결과를 입력받아 특정 작업을 수행한다.
  *   반환 타입이 CompletableFuture\<Void>로 바뀐다.

      > Function: `T → void`
* **runAfterEither**
  * CompletionStage, Runnable 객체를 인자로 받는다.
  * CompletableFuture\<T>, CompletionStage\<?> 중 먼저 `complete`된 후 특정 작업을 수행한다.
  *   반환 타입이 CompletableFuture\<Void>로 바뀐다.

      > Function: `() → void`

***

* **getNumberOfDependents**
  * 현재 stack에 등록된 Completion 객체의 개수를 반환한다. Completion 객체에 체이닝되어 있는 것까지 모두 확인하여 반환한다.
  * 콜백 뿐만 아니라 get()으로 결과를 기다릴 때에도 Completion을 stack에 추가할 수 있다.
