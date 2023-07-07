# Process Scheduling

### CPU burst vs I/O burst

* 프로세스는 CPU명령 수행하고, I/O 수행하는 과정이 반복된다.
* CPU burst
  * CPU가 명령어를 수행하는 구간, 프로세스가 Running인 상태
  * CPU-bound process: CPU burst가 큰 프로세스 ex) 슈퍼컴퓨터에서 실행되는 기상예측, 화학 분자 프로그램
* I/O burst
  * I/O의 수행이 끝나기를 기다리는 구간
  * 프로세스가 Waiting인 상태
  * Ready 상태는 나타나지 않음
  * I/O-bound process: I/O burst가 큰 프로세스 ex) 문서 편집기, 동영상 재생
* 대부분 우리는 I/O-bound process를 사용한다.

### Preemptive vs Non-Preemptive

* Preemptive scheduling
  * 더 중요한 프로세스를 먼저 처리하는 방식
  * 성능이 훨씬 좋고 응답속도가 빠르다.
* Non-preemptive scheduling
  * 실행 중인 프로세스보다 더 중요한 프로세스가 들어와도 양보하지 않고, 원래 프로세스가 waiting / ready / terminated 될 때 까지 수행

### 스케줄링 방식

#### FCFS/FIFO

* First-Come, First-Served Scheduling
* 일찍 도착하는 순서대로 스케줄링
* non-preemptive 방식 -> 공평하게 다뤄지므로 starvation이 없음
* convey effect: 실행시간이 적은 프로세스가 긴 프로세스 뒤에 있을 경우 average waiting time이 커질 수 있다.
* Burst time이 오래 걸리더라도 먼저 들어왔으면 그것부터 실행되므로 waiting time이 점점 길어지게 된다.
* 만약 burst time이 짧은 프로세스를 우선 수행한다면 waiting time이 현저히 줄어듦

#### SJF(Shortest-Job-First) Scheduling

* Interactive system에서는 waiting time을 줄여야 사용자 만족도를 높일 수 있음
* burst time이 가장 짧은 프로세스부터 실행하는 방식
* 주어진 프로세스들의 average waiting time을 가장 작게 만듦
* Example of Non-Preemptive SJF: 프로세스가 실행 중일 때 Burst time이 작은 프로세스가 들어와도 양보해주지 않음, 프로세스 수행이 끝난 후 여러 프로세스가 대기중(P1이 실행되는 중에 P2, P3, P4가 모두 도착)일 때 burst time이 적은 순서대로 수행됨
* Example of Preemptive SJF: 남은 작업 시간(burst time) 중 가장 작은 것이 먼저 수행, 프로그램 실행 중 계속해서 burst time을 비교해서 가장 적은 것으로 수행
* SJF 방식은 CPU burst time을 정확히 예측하기 어렵기 때문에 실제 컴퓨터 시스템에 적용하기 어려움

#### Priority Scheduling

* 프로세스의 의미에 따라 우선순위 부여하는 방식
* 우선순위가 낮은 프로세스는 계속 실행되지 않는 starvation 발생
* aging을 통해 오래 기다린 프로세스의 우선순위를 높여줌

#### Round Robin (RR)

* time quantum(small unit of CPU time, usually 10-100 milliseconds)에 따라 번갈아가면서 프로세스 수행
* 같은 레이어의 프로세스는 fair하게 수행, waiting time, response time 줄여줌
* a rule of thumb: 80%의 프로세스의 CPU burst가 time quantum보다 작아야 함
* time quantum에 따른 trade-off 발생: time quantum이 작으면 context switch 자주 발생해 오버헤드 가 커져 response time 증가 -> 적절히 조정하는 것이 중요
