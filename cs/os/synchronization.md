# Synchronization

## synchronization

{% hint style="info" %}
여러 개의 프로세스에서 공유 자원을 접근 시 접근 순서에 따라 결과값이 다르게 출력될 수 있다. 이를 방지하기 위해 여러 프로세스/스레드를 동시에 실행해도 공유 데이터의 일관성을 유지하도록 동기화하는 과정이 필요하다.
{% endhint %}

#### race condition

* 여러 프로세스/스레드가 동시에 같은 데이터를 조작할 때, 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황

#### critical section

* race condition을 피하기 위해 이 영역을 두어 관리한다.
* 공유 데이터의 일관성을 보장하기 위해 하나의 프로세스/스레드만 진입해 실행 가능한 영역
* 하나의 프로세스/스레드만 진입해 실행하는 것을 mutual exclusion이라고 한다
* lock을 사용해 mutual exclusion을 구현할 수 있다

#### Solution to Critical-Section Problem

1. Mutual Exclusion
   * 어떤 프로세스가 critical section에서 작업할 경우 다른 프로세스의 접근을 막음
2. Progress
   * critical section에 현재 프로세스가 없는 경우 새로운 프로세스의 작업을 진행시킴
3. Bounded Waiting
   * critical section에 들어가려는 프로세스가 무한히 대기하고 있으면 안됨

## Mechanisms for Critical Sections

1. Lock
   * 프로세스가 critical section에 들어가면 lock, 프로세스가 끝나면 unlock하는 단순한 방식
2. Semaphores
   * 변수 하나를 만들어 critical section 진입할 프로세스/스레드를 관리하는 방식
3. Monitors
   * accuracy, condition value, signal과 waiting을 사용해 관리

> atomic 명령어: 실행 중간에 간섭받거나 중단되지 않는다. 그리고 같은 메모리 영역에 대해 동시에 실행되지 않는다.

### 스핀락

* **다른 스레드가 lock을 소유하고 있다면 그 lock이 반환될 때까지 계속 확인하며 기다리는 것**
* 무한 루프를 돌면서 다른 스레드에게 CPU를 양보하지 않는 것
* critical section에서의 작업이 context switching보다 빨리 끝나거나(즉, 락을 잠깐 동안만 유지하는 경우), 락이 드문 경우 스핀락이 뮤텍스보다 효율적이다.
* single core인 경우 유용하지 않다.

```cpp
volatile int lock = 0; // global variable

int test_and_set(int* lockPtr) {
	int oldLock = *lockPtr;
	*lockPtr = 1;
	return oldLock; // lock을 1로 바꿔주고 기존 값 리턴
}

void critical() {
  // 기존 lock 값이 1이면 계속 기다리고, 기존 lock값이 0이면 1로 변경 후 critical section 접근
	while(test_and_set(&lock) == 1);
	// critical section 작업 진행
	lock = 0;
}
```

### 뮤텍스

* 락이 준비되면 깨우는 방식
* 락을 걸은 스레드만이 critical section을 빠져나갈 때 락을 해제할 수 있다.

```cpp
class Mutex {
	int value = 1;
	int guard = 0;
}
```

* lock()
  * critical section에 들어가기 위해 guard를 확인하고, 다른 스레드가 value를 사용하지 않음을 확인한다
  * value 가 0이면 대기 모드
  * value가 1이면 value와 guard를 0으로 두어 락을 걸고 critical section으로 진입한다.
* unlock()
  * 큐에 대기중인 스레드가 있다면 깨우고, 없다면 value를 1로 두어 락을 해제한다.
* guard에 의해 보호받는 critical section에서 동작하는 코드는 짧은 시간으로 끝나는 코드이므로, 스핀락보다 CPU 낭비가 적다.
* 상호 배제만 필요하다면 사용해야 함

### 세마포어

* signal mechanism을 가진다. 즉 변수를 가지고 접근 여부를 판별한다.
* 하나 이상의 프로세스/스레드가 critical section에 접근 가능하도록 하는 장치

```cpp
class Semaphore {
	int value = 1;
	int guard = 0;
}

Semaphore::wait() {
	if (value == 0) {
		// 현재 스레드를 대기 큐에 입력
	} else {
		value -= 1; // critical section에 하나의 스레드가 들어갔다는 의미로 감소시킴
	}
}

Semaphore::signal() {
	if(!대기큐.isEmpty()) {
		// 대기 큐의 스레드 하나 활성화
	} else {
		value += 1; // critical section에 접근가능한 스레드가 늘었다는 의미로 증가시킴
	}
}

void main() {
	semaphore -> wait();
	// critical section 들어가 작업
	semaphore -> signal();
}
```

* wait()
  * 자원을 획득하는 연산
  * value가 0이 넘어가기를 기다린 후 넘어가면 1 감소 시킨다.
* signal()
  * 자원을 해제하는 연산

#### 세마포어 종류

* 바이너리 세마포어
  * 최대 1의 value 값을 가질 수 있다.
  * 뮤텍스와 다른 점
    * 뮤텍스는 락을 가진 자만 락을 해제할 수 있지만 세마포어는 락을 가지지 않아도 락을 해제할 수 있다.
    *   뮤텍스는 priority inheritance 속성을 가지지만, 세마포어는 그 속성이 없다.

        > priority inheritance: 우선순위가 높은 작업이 우선순위가 낮은 작업이 Lock을 잡고 있으면 기다려야 한다. 따라서 우선순위가 낮은 작업이더라도 우선순위를 높여 먼저 완료해버린 후 기다리던 작업을 완료해야 한다.

        [https://j-i-y-u.tistory.com/21](https://j-i-y-u.tistory.com/21)
*   카운팅 세마포어

    * 2개 이상의 signal 값을 가질 수 있다.
    * 순서를 정해줄 때 사용 가능하므로 작업 간 실행 순서 동기화가 필요하다면 사용해야 함
    * 아래와 같이 프로그래밍한다면 플로우는?
      1. task 1 과 task 2를 각각의 스레드에서 실행한다.
      2. task 2 완료된 상태에서 task1 완료를 기다린 후, 완료되었다면 task 3을 실행한다.

    ```cpp
    // thread 1
    <task 1>
    semaphore -> signal()

    // thread 2
    <task 2>
    semaphore -> wait()
    <task 3>
    ```

### 모니터

* mutual exclusion을 보장한다.
* 조건에 따라 스레드가 대기 상태로 전환 가능하다.
* 한 번에 하나의 스레드만 실행되야 할 때, 여러 스레드와 협업이 필요할 때 사용할 수 있다.
* mutex, condition variables로 구성된다.

#### mutex

* critical section에 진입하기 위해 mutex lock을 취득해야 한다.
* mutex lock을 취득하지 못한 스레드는 큐에 들어간 후 대기 상태로 전환
* mutex lock을 쥐고 있던 스레드가 lock을 반환하면 대기 큐에 있던 스레드 중 하나가 실행된다.

#### condition variables

* 대기 큐(waiting queue)를 가진다.
* 조건이 충족되길 기다리는 스레드들이 대기 상태로 머무는 곳
* wait()
  * 스레드가 자기 자신을 conditino variable의 대기 큐에 넣고 대기 상태로 전환한다.
* signal()
  * 대기 큐에 있는 스레드 중 하나를 깨운다.
* broadcast()
  * 대기 큐에 있는 모든 스레드를 깨운다.





