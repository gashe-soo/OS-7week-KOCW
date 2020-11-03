# CH 5. CPU Scheduling

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.

<br/><br/>

### CPU And I/O Bursts in Program execution

<img src="https://raw.githubusercontent.com/zoomKoding/zoomKoding.github.io/source/assets/_posts/os5-1.png" alt="OS 기본개념 정리) CH5 Process Scheduling" style="zoom:50%;" />

- CPU를 사용하는 단계 : CPU burst

- I/O를 사용하는 단계 : I/O burst

<br/>

프로세스는 특성에 따라 두 가지로 나뉜다.

| I/O bound process                                    | CPU bound process                          |
| ---------------------------------------------------- | ------------------------------------------ |
| I/O에 많은 시간이 필요한 job = many short CPU bursts | 계산 위주의 job = few very long CPU bursts |

=> 여러 종류의 job이 섞여 있기 때문에 **CPU 스케줄링이 필요**하다.

- Interactive job에게 적절한 response 제공이 필요하다. 

- CPU와 I/O 장치 등 시스템 자원을 골고루 효율적으로 사용 => 효율성 + 형평성 고려 필요

<br/>

### CPU Scheduler & Dispatcher

CPU Scheduler = OS 내의 코드

- Ready 상태의 프로세스 중에서 CPU를 할당할 프로세스 고른다.

Dispatcher : CPU의 제어권을 CPU Scheduler에 의해 선택된 프로세스에게 넘긴다

- Context Switch
- User mode로 전환
- user program의 적절한 위치로 jump

<br/>

CPU 스케줄링이 필요한 경우?

1. Running -> Blocked (EX. I/O system call)

2. Running -> Ready (EX. timer interrupt)

3. Blocked -> Ready (EX. I/O complete interrupt)

4. Terminate

> 1, 4에서의 scheduling은 nonpreemptive = 자진 반납
>
> 나머지는 preemptive = 강제로 빼앗음

<br/>

### Scheduling Criteria

시스템 관점

- CPU utilization(이용률) =놀지 않고 일해라
- Throughput(처리량) = 주어진 시간에 몇 개의 일을 처리하는가

프로세스 관점

- Turnaround time(소요시간, 반환시간) = CPU 들어와서 나갈 때까지 걸린 시간
- Waiting time(대기 시간) = CPU를 쓰려고 기다린 시간
- Response time(응답 시간) = 최초로 CPU를 얻는 시간

<br/>

### Scheduling Algorithms

#### FCFS(First-Come First-Served)

선입선출 알고리즘 = 들어온 순서대로 CPU를 사용한다.

| process | burst time |
| ------- | ---------- |
| P1      | 24         |
| P2      | 3          |
| P3      | 3          |

만약 P1,P2,P3 순서대로 온다면 간트 차트는 아래와 같다.

![week_1_ch2_img1.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week3_ch5_img1.jpg?raw=true)

waiting time의 평균은 (0+24+27)/3 = 17이다.

반면 P2,P3,P1의 순서대로 온다면 waiting time의 평균은 (6+0+3)/3 = 3이다.

![week_1_ch2_img1.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week3_ch5_img2.jpg?raw=true)

<br/>

이처럼 만약 Time이 긴 프로세스가 먼저 CPU를 사용하면 cpu를 짧게 쓰는 job들은 오래 기다려야 한다.

=> 비효율적

> Convey Effect(호위 효과)
>
> Short process behind Long process
>
> 만약 하나의 CPU bound process와 여러 개의 I/O bound process를 가진다고 가정하자.
>
> 1. 처음 CPU bound process가 CPU를 할당받는다고 가정한다.
> 2. 그렇다면 I/O bound process들은 준비 큐에서 기다리고 있고 I/O 장치들은 쉬고 있다.
> 3. CPU bound process가 cpu burst를 끝내고 cpu를 넘겨준다.
> 4. I/O bound process들은 짧은 cpu burst를 끝내고 I/O 처리하러 간다.
>
> 위에서 보이는 문제점은 무엇인가? 
>
> I/O bound process들은 긴 I/O 시간이 걸린다. 그렇기에 빠르게 cpu burst를 끝내고 I/O을 하러 가야 한다.
>
> 그런데, CPU bound process가 먼저 긴 cpu burst를 하고 있어서 I/O도 처리하러 가지 못하고 기다리게 된다.
>
> 이처럼 모든 다른 프로세스들이 하나의 긴 프로세스가 CPU를 양도하기를 기다리는 것을 호위 효과라 한다.

<br/>

#### SJF(Shortest-Job First)

SJF 알고리즘은각 프로세스의 다음번 CPU burst time을 가지고 스케줄링에 활용한다.

- 기준 : CPU의 이용이 가능할 때, CPU burst time이 가장 짧은 프로세스를 제일 먼저 스케줄한다.
- SJF는 Non preemptive 비선점 알고리즘이다. 
  - 일단 CPU를 잡으면 CPU burst가 완료될 때까지 CPU를 선점당하지 않음



예시를 통해 이해하자.

| process | arrival time | burst time |
| ------- | ------------ | ---------- |
| p1      | 0            | 6          |
| p2      | 0            | 8          |
| p3      | 0            | 7          |
| p4      | 0            | 3          |

위와 같이 모든 프로세스가 동시에 도착했다. 그러나 burst time이 다르기 때문에 순서도 달라진다.

1) CPU를 사용하는 시점에 p4가 3으로 제일 짧다. 따라서 가장 먼저 CPU를 사용한다.

2) p4가 끝난 시점에는 p1이 다음으로 짧다. p1이 사용이 끝나면 p3,p2 순이다.

![week_1_ch2_img1.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week3_ch5_img3.jpg?raw=true)

각 프로세스의 대기 시간은 (3+16+9+0)/4 = 7이다. 

- SJF 알고리즘은 최적이다.
  - 주어진 프로세스들에 대해 minimum average waiting time을 보장(SRTF가 더 짧다.)
  - 짧은 프로세스들이 긴 프로세스의 앞으로 이동하여 대기 시간이 줄기 때문이다.
- 문제점
  - 극단적으로 cpu burst time이 짧은 process를 선호 => 긴 프로세스는 계속 할당받지 못한다 = starvation
  - cpu burst time을 모두 알 수 없다. => 시스템에서 사용할 수 없으나 burst time을 과거의 통계로 추정(estimate) = exponential averaging

<br/>

#### SRTF(Shorest Remaining Time First)

SJF 알고리즘은 비선점형 알고리즘이다. 

반면에 SRTF는 선점형 알고리즘이다.

- 현재 수행 중인 남은 burst time보다 더 짧은 cpu burst time을 가지는 새로운 프로세스가 도착하면 CPU를 뺏는다.

예시를 보자.

| process | arrival time | burst time |
| ------- | ------------ | ---------- |
| p1      | 0            | 8          |
| p2      | 1            | 4          |
| p3      | 2            | 9          |
| p4      | 3            | 5          |

![week_1_ch2_img1.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week3_ch5_img4.jpg?raw=true)

- P1이 0초에 도착한다. 아무도 사용하지 않으므로 p1이 cpu를 사용한다.
- 1초에 p2가 도착한다. p2의 남은 시간은 4초인 반면, p1은 7초가 남았기에 더 짧은 p2에 cpu를 할당한다.
- 2초에 p3가 도착했지만 p2는 3초, p3는 9초 남았으므로 p2가 계속 사용한다.
- 3초에도 동일하게 p2가 사용한다.
- 5초에 p2가 모두 사용했을 때 남은 시간이 가장 짧은 것은 p4이므로 p4가 5초를 사용한다.
- 10초에 p1의 남은 시간이 p3보다 짧다. 따라서 p1,p3 순으로 cpu를 할당받는다.

평균 대기시간은 (9 + 0+2+15)/4 = 6.5초이다.

<br/>

#### Priority Scheduling

Highest Priority를 가진 프로세스에게 CPU를 할당한다.

- smallest integer = highest priority

- SJF는 일종의 priority scheduling이다.
- priority scheduling은 선점형, 비선점형 둘 다 가능하다.
- 문제점
  - Starvation : 낮은 우선 순위의 프로세스는 실행되지 않는다.
- 해결방안
  - Aging : 시간이 진행할수록 프로세스의 우선순위를 올려준다.

<br/>

#### Round Robin(RR)

- 각 프로세스는 동일한 크기의 할당 시간(time quantum or time slice)을 가진다.
- 할당 시간이 지나면 **프로세스는 선점당하고** ready queue의 맨 뒤에 들어간다.
- n개의 프로세스가 ready queue에 있고 할당 시간이 q time unit인 경우 어떤 프로세스도 (n-1)*q time unit이상 기다리지 않는다.
- Performance
  - 시간 할당량이 커지면? => FIFO(FCFS)
  - 시간 할당량이 작아지면? => context switch의 오버 헤드가 커진다.
- 일반적으로 SJF보다 average turnaround time이 길지만, response time은 더 짧다.

<br/>

#### Multilevel Queue

![Multilevel Queue Scheduling Algorithm | Studytonight](https://www.studytonight.com/operating-system/images/multi-level-scheduling-1.png)

- Ready queue를 여러 개로 분할
  - foreground(interactive)
  - background(batch- no human interaction)
- 각 큐는 독립적인 스케줄링 알고리즘을 가짐
  - foreground = RR
  - background = FCFS
- 큐에 대한 스케줄링이 필요
  - Fixed Priority scheduling
    - serve all foreground, then serve background
    - starvation 가능성
  - Time slice
    - 각 큐에 cpu time을 적절한 비율로 할당



<br/>

#### Multilevel Feedback Queue

![Multilevel Feedback Queue Scheduling (MLFQ) CPU Scheduling - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/Multilevel-Feedback-Queue-Scheduling-300x269.png)

- 프로세스가 다른 큐로 이동 가능하다.
- 에이징을 feedback queue으로 구현할 수 있다.
- Multilevel-feedback-queue scheduler를 정의하는 파라미터들
  - queue의 수
  - 각 큐의 scheduling algorithm
  - process를 상위 큐로 보내는 기준
  - process를 하위 큐로 내쫓는 기준
  - 프로세스가 CPU 서비스를 받으려 할 때 들어갈 큐를 결정하는 기준

- 프로세스는 처음 높은 우선순위 큐에 할당 -> 갈수록 낮은 우선순위 큐로 이동한다.
- 우선순위가 낮아질수록 시간 할당이 길어진다.

<br/>

### 기타 스케줄링 (참고사항)

#### Multiple Processor Scheduling

- CPU가 여러 개인 경우 스케줄링은 더욱 복잡해진다.
- Homogeneous processor인 경우
  - 큐에 한 줄로 세워서 각 프로세서가 알아서 꺼내가게 할 수 있다.
  - 반드시 **특정 프로세서에서 수행되어야 하는 프로세스가 있는 경우에는 문제가 더 복잡해짐**
- 여러 개의 코어로 Load Sharing이 가능해진다.
  - 일부 프로세서에 job이 몰리지 않도록 부하를 적절히 공유하는 메커니즘 필요
  - 별개의 큐를 두는 방법 vs 공동 큐를 사용하는 방법
- Symmetric Multiporcessing (SMP)
  - 각 프로세서가 각자 알아서 스케줄링 결정
  - 공통 준비 큐 vs 프로세서별 준비 큐
    - 공통 준비 큐는 queue access때문에 locking이 필요하고 이로 인해 성능의 병목이 일어날 수 있다.
    - 프로세서별 준비 큐는 가장 일반적인 접근 방식
- Asymmetric Multiprocessing
  - 하나의 프로세서가 시스템 데이터의 접근과 공유를 책임지고 나머지 프로세서는 거기에 따름
  - 문제점
    - 마스터 서버가 전체 시스템 성능을 저하할 수 있는 병목이 된다.

<br/>

#### Real-Time Scheduling

- Hard real-time systems
  - Hard real-time task는 정해진 시간 안에 반드시 끝내도록 스케줄링해야 한다.
  - 미리 설정한다.
- Soft real-time computing
  - 일반 프로세스에 비해 높은 priority를 갖도록 해야 한다.

<br/>

#### Thread Scheduling

- Local Scheduling
  - User level thread의 경우 사용자 수준의 thread library에 의해 어떤 thread를 스케줄할지 결정
- Global Scheduling 
  - Kernel level thread의 경우 일반 프로세스와 마찬가지로 커널의 단기 스케줄러가 어떤 thread를 스케줄할지 결정

<br/>

### Algorithm Evaluation

- Queueing models
  - 확률 분포로 주어지는 arrival rate와 service rate 등을 통해 각종 performance index값을 계산
- Implementation & Measurement
  - **실제 시스템**에 알고리즘을 구현하여 **실제 작업(workload)**에 대해서 성능을 측정 비교 
- Simulation
  - 알고리즘을 **모의 프로그램**으로 작성 후 trace를 입력으로 하여 결과 비교





### REFERENCE

[zoomKoding.github.io](https://zoomKoding.github.io)

