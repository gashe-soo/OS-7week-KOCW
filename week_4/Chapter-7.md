# CH 7. Deadlocks

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.

<br/><br/>

### Deadlock Problem

#### Deadlock이란?

- 일련의 프로세스들이 서로가 가진 자원을 기다리며 block된 상태

<br/>

#### Resource

- 하드웨어, 소프트웨어 등을 포함하는 개념
  - EX. I/O device, CPU, memory space, semaphore etc
- 프로세스가 자원을 사용하는 절차
  - Request, Allocate, Use, Release

<br/>

#### Deadlock Example

- Ex 1.
  - 시스템에 2개의 tape drive가 있다.
  - 프로세스 P1,P2 각각이 하나의 tape drive를 보유한 채 다른 하나를 기다리고 있다.
- Ex 2.
  - Binary semaphores A and B
  - Dining Philosopher's problem

<br/>

### Deadlock의 조건

Deadlock은 아래의 **4조건을 동시에 성립**될 때 발생한다. 

**Mutual Exclusion(상호 배제)**

- 매 순간 하나의 프로세스만이 자원을 사용할 수 있다.

**No Preemption(비선점)**

- 프로세스는 자원을 스스로 내어놓을 뿐 강제로 빼앗기지 않는다.

**Hold and Wait(보유 대기)**

- 자원을 가진 프로세스가 다른 자원을 기다릴 때 보유 자원을 놓지 않고 계속 가지고 있다.

**Circular Wait(순환 대기)**

- 자원을 기다리는 프로세스간에 사이클이 형성되어야 한다.

- 프로세스 P0,P1...Pn이 있을 때 P0은 P1이 가진 자원을, P1은 P2가 가진 자원을...Pn은 P0이 가진 자원을 기다린다.

<br/>

### Resource-Allocation Graph

Deadlock은 Resource Allocation Graph(자원 할당 그래프)라는 방향 그래프로 정확하게 나타낼 수 있다.

<br/>

표시는 다음과 같다

- 프로세스는 P, 자원은 R이라 표현한다.
- 자원은 여러 인스턴스를 가질 수 있고 사각형 내 점은 인스턴스를 뜻한다.
- 프로세스 P가 자원 R의 인스턴스를 하나 요청하는 것을 P -> R 로 표현한다. 이는 요청 간선이라 한다.
- 자원 R의 한 인스턴스가 프로세스 P에 할당된 것은 R -> P로 표현한다. 이는 할당 간선이라 한다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img1.jpg?raw=true" alt="week4_ch7_img1.jpg" style="zoom: 80%;" />

위의 그림을 해석해보자.

1. P1은 R2 중 하나의 인스턴스를 점유하고 있다. 또한, R1을 요청하며 대기한다.
2. P2는 R1,R2의 인스턴스를 하나씩 점유하고, R3을 요청하며 대기한다.
3. P3는 R3의 인스턴스를 점유하고 있다.

<br/>

자원 할당 그래프는 deadlock를 표현한다고 했다. 여기서 보이는 특성이 있다.

- 그래프에 cycle이 없으면 deadlock이 아니다.
- 그래프에 cycle이 있을 때
  - 자원당 하나의 인스턴스만 있다면 => deadlock

  - 여러 인스턴스가 있다면 => deadlock의 가능성이 있다. 아닐 수도 있다.

  - <img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img2.jpg?raw=true" alt="week4_ch7_img2.jpg" style="zoom: 80%;" /> 

     deadlock 

  - <img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img3.jpg?raw=true" alt="week4_ch7_img3.jpg"  /> 

    -  no deadlock



<br/>

### Deadlock의 처리 방법

**Deadlock Prevention**

- 자원 할당 시 Deadlock의 4가지 필요 조건 중 어느 하나가 만족되지 않도록 하여 예방한다.

**Deadlock Avoidance**

- 자원 요청에 대한 부가적인 정보를 이용해서 deadlock의 가능성이 없는 경우에만 자원을 할당한다. = 가능성을 회피한다.
- 시스템 state가 원래 state로 돌아올 수 있는 경우에만 자원 할당한다.

**Deadlock Detection and recovery**

- Deadlock 발생은 허용하되 그에 대한 detection 루틴을 두어 feedback 발견시 recovery 실행

**Deadlock Ignorance**

- Deadlock을 시스템이 책임지지 않는다.
- Unix를 포함한 대부분의 OS가 채택하고 있다.



<br/>

### Deadlock Prevention

**Mutual Exclusion**

- 공유 가능한 자원들을 사용하면 배타적인 접근을 요구하지 않으므로 교착상태가 발생하지 않는다.

- 그러나 어떤 자원들은 근본적으로 공유가 불가능하다. ex)mutex lock

  => 따라서 상호배제 조건을 성립시키지 못하기는 어렵다.

**Hold and Wait**

- 프로세스가 **자원을 요청할 때 다른 어떤 자원도 가지고 있지 않아야 한다**.

- 방법 

  1. 프로세스 시작 시 모든 필요한 자원을 할당받게 하는 방법 => 다만 자원 요청의 동적 특성으로 비효율적이다.

  2. 자원이 필요할 경우 점유하던 자원을 모두 놓고 다시 요청한다.

**No Preemption**

- **process가 어떤 자원을 기다려야 하는 경우 이미 보유한 자원이 선점될 수 있다**. 즉, 다른 프로세스들이 사용할 수 있다.
- 모든 필요한 자원을 얻을 수 있을 때 그 프로세스는 다시 시작된다.
- State를 쉽게 save하고 restore할 수 있는 자원에서 주로 사용(CPU, memory) 
  - CPU,memory는 선점당해도 주소를 기억하고 있기 때문.

**Circular wait**

- 모든 자원 유형에 **할당 순서를 정하여 정해진 순서대로만 자원을 할당**한다
- Ex. 순서가 3인 자원 R1를 보유 중인 프로세스가 순서가 1인 자원 R2를 할당받기 위해서는 우선 R1를 release해야 한다.

<br/>

**Deadlock Prevention의 문제점**

Utilization 저하, Throughput 감소, Starvation 문제

<br/>

### Deadlock Avoidance

- 자원 요청에 대한 부가정보를 이용해서 자원 할당이 deadlock으로부터 안전한지를 동적으로 조사해서 안전한 경우에만 할당한다.
- 가장 단순하고 일반적인 모델은 **프로세스들이 필요로 하는 각 자원별 최대 사용량을 미리 선언**하도록 하는 방법이다.
  - 이를 통해 deadlock이 발생하는 경우를 피해간다.

<br/>

회피한다는 뜻은 Deadlock으로부터 안전하다고 말할 수 있다. 이것을 safe state라고 한다.

**Safe state**이란? 

시스템 내의 프로세스들에 대한 safe sequence가 존재하는 상태를 뜻한다. 

즉, **deadlock을 만들지 않고 프로세스들이 요청하는 자원을 sequence대로 할당해 줄 수 있다**는 것이다.

<br/>

그렇다면 **safe sequence**는 어떤 경우에 존재한다고 말할 수 있을까?

- 프로세스의 sequence가 safe하려면 **Pi의 자원 요청이 "가용 자원+ 종료된 Pj 의 보유 자원"에 의해 충족**되어야 한다.
- 조건을 만족하면 다음 방법으로 모든 프로세스의 수행을 보장한다.
  - Pi의 자원 요청이 즉시 충족될 수 없으면 모든 Pj(j<i)가 종료될 때까지 기다린다.
  - P(i-1)이 종료되면 Pi의 자원요청을 만족시켜 수행한다.

설명이 너무 어렵다. 즉 **Pi의 요청한 자원이 (남아있던 자원 + 다른 프로세스에서 반납된 자원)보다 작으면 safe하고 아니면 unsafe한 것이다.**

그래서 safe하다면 Pi을 수행하고 Pi의 자원을 반납하는 것이다. 이렇게 모든 프로세스에 대하여 수행한다.

<br/>

**그래서 결론은?**

- 시스템이 safe state에 있으면  => no deadlock
- 시스템이 unsafe state에 있으면 => possibility of deadlock
- **Deadlock Avoidance이란**
  - **시스템이 unsafe state에 들어가지 않는 것을 보장해야 한다.**
  - 2가지 알고리즘
    - 자원당 인스턴스 하나라면 => Resource Allocation Graph Algorithm 사용
    - 자원당 여러 인스턴스라면 => Banker's Algorithm 사용

<br/>

#### Resource Allocation Graph Algorithm

Resource Allocation Graph(자원 할당 그래프)를 다시 사용해보자.

요청 간선, 할당 간선에 **예약 간선**(claim edge)이라는 것을 추가한다.

예약 간선 P->R은 **프로세스 P가 미래에 자원 R을 요청할 것**이라는 의미다. 이는 요청 간선과 유사하지만 **점선으로 표시**한다

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img4.jpg?raw=true" alt="week4_ch7_img4.jpg"  />

그림과 같이 점선으로 표시해주면 된다. 

또한, 중요한 것은 **인스턴스가 하나이므로 사이클이 발생하면 안된다.**

이제 그래프를 보자. 

P2가 R2를 요청한다고 가정하자. 현재 R2는 가용 상태이지만, 만약 P2에 할당하게 되면 아래와 같이 사이클이 발생할 수 있다. 

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img5.jpg?raw=true" alt="week4_ch7_img5.jpg"  />



그렇기 때문에 우리는 사이클 탐지 알고리즘을 이용해 안전성을 검사한다.

만약 사이클이 발견되면 사이클을 발생하는 요청을 대기하게 한다. 

그림에서는 R2를 P2가 아닌 P1에 할당하게 한 후, P1이 종료되면 반납된 자원을 P2에 할당하는 것이다.

<br/>

#### Banker's Algorithm

회피하기 위해서 **Pi의 요청한 자원이 (남아있던 자원 + 다른 프로세스에서 반납된 자원)보다 작으면 safe하고 아니면 unsafe한 것이다**라고 했다.

<br/>

먼저, Banker's Algorithm의 과정과 필요한 자원 상태이다.

0. 전제: 프로세스는 현재 이미 자원을 할당받았다.(이하 allocation)
1. 프로세스마다 시작할 때, 요청할 자원의 최대 개수(max)를 자원 종류마다 선언한다. (물론 자원 보유 수를 넘어가면 안된다.)
2. 자원의 현재 가능한 수(Avaliable)  = 자원 총 보유 수 - 현재 할당된 수이다. 
3. 앞으로 프로세스가 요청할 자원의 수(need)는 max - allocation이다. 
4. 항상 최악의 경우를 가정하기 때문에 available > need인 경우에만 요청을 받아들인다.

<br/>

사실 말로 설명하는 것이 더 어려우니 예시를 통해 이해해보자.

5개의 프로세스 P(0~4)와 자원 A,B,C가 있다고 가정한다. A는 10개, B는 5개, C는 7개가 있다고 가정하자.

자원의 현재 상태는 아래와 같다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img6.jpg?raw=true" alt="week4_ch7_img6.jpg" style="zoom: 80%;" />

프로세스의 자원 요청은 need = max - allocation이므로 다음과 같을 것이다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img7.jpg?raw=true" alt="week4_ch7_img7.jpg" style="zoom: 80%;" />

그러면 알고리즘에 의해서 안전한지 확인해보자.

- P0가 현재 사용 가능한 자원은 (A,B,C) = (3,3,2)이다. 그런데 최대 요청은 (7,4,3)까지 가능하다. (물론 이보다 적게 요청할 수 도 있지만, 회피알고리즘은 최악을 피해야 한다.) 따라서 need > available이므로 불가능하다.
- P1 차례다. 사용 가능한 자원은 그대로 (3,3,2)이다. P1의 need는 (1,2,2)이다. need < available이므로 가능하다. 따라서 P1에 자원을 할당한 후 P1이 종료되면서 자원을 반납한다.
- P2 차례다. 현재 가능한 자원은 (3,3,2) + (2,0,0) = (5,3,2)이다. P2의 need는 (6,0,0)이다. A에서 need > available이므로 할당 불가하다. 넘어간다.
- P3 차례다. available은 (5,3,2)이다. P3의 need는 (0,1,1)이다. need < available이므로 자원 할당 가능하다.
- P4 차례다. available은 (5,3,2) + P3의 (2,1,1) = (7,4,3)이다. P4도 가능하다.

이렇게 계속 반복하면 순서는 <P1,P3,P4,P0,P2>로 safe sequence가 된다.

<br/>

회피 알고리즘은 자원이 남아있는 데도 요청을 안 받아들이므로 사실 비효율적이다. 



<br/>

### Deadlock Detection and Recovery

#### Deadlock Detection

- 자원당 하나의 인스턴스인 경우
  - 자원할당 그래프에서의 cycle은 곧 deadlock를 의미한다.
- 자원당 여러 개의 인스턴스인 경우
  - Banker's Algorithm과 유사한 방법 활용

<br/>

#### Wait-for graph 알고리즘

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch7_img8.jpg?raw=true" alt="week4_ch7_img8.jpg"  />

- 자원당 하나의 인스턴스인 경우, graph에서 자원을 빼버리고 프로세스로만 구성
- Wait-for graph
  - 자원할당 그래프의 변형
  - 프로세스만으로 node 구성
  - Pj가 가지고 있는 자원을 Pk가 기다리는 경우 Pk->Pj
- Algorithm
  - Wait-for graph에 사이클이 존재하는 지를 주기적으로 조사
  - O(n^2)

<br/>

#### Recovery

Deadlock에서 회복할 때, 두 가지 방법을 사용한다.

1. **Process termination (프로세스 종료)**

- Abort all deadlocked processes (deadlock 프로세스를 모두 중지)
- Abort one process at a time until the deadlock cycle is eliminated (deadlock이 제거될 때까지 한 프로세스씩 중지)

다만, 프로세스를 중지하는 것은 어렵다. 데이터 갱신 도중 종료된다면 데이터 무결성은 보장하기 어렵기 때문이다.

또한, 어떤 프로세스를 중지할 것인지 결정해야 한다. 즉, 비용을 비교해야 한다.

결정할 요인은 다음과 같이 많다.

- 프로세스의 우선순위
- 지금까지 프로세스가 수행된 시간과 지정된 일을 종료하는 데 더 필요한 시간
- 프로세스가 사용한 자원의 유형과 수
- 프로세스가 종료하기 위해 더 필요한 자원의 수
- 얼마나 많은 수의 프로세스가 종료되어야 하는지 

<br/>

2. **Resource Preemption (자원 선점)**

교착상태가 제거될 때까지 자원을 선점하여 다른 프로세스에 주어야 한다.

- selection of victim 
  - 비용을 최소화할 victim을 선정한다.
  - 선점의 순서를 정해야 한다.
- Rollback
  - safe state로 rollback하여 process를 restart한다.
- Starvation 문제
  - 동일한 프로세스가 계속해서 victim으로 선정되는 경우
  - cost factor에 rollback 횟수도 같이 고려

<br/>

### Deadlock Ignorance

Deadlock이 일어나지 않는다고 생각하고 아무 조치도 취하지 않는다.

- Deadlock이 드물게 발생하므로 deadlock에 대한 조치 자체가 더 큰 overhead일 수 있다.
- 만약, 시스템에 deadlock이 발생한 경우 시스템이 비정상적으로 작동하는 것을 사람이 느낀 후 직접 process를 죽이는 등의 방법으로 대처
- 대부분의 OS가 채택

<br/><br/>