# CH 6. Process Synchronization

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.



### Process Synchronization?

Process Synchronization은 무엇이고 왜 필요한가?



computer란 data를 storage에서 가져와서 compute(연산)을 한 후에 다시 storage에 저장한다.

그런데 만약 data를 동시에 여러 곳에서 접근해서 compute한다면? 

예를 들어 p1은 data를 1 증가 시키고 저장한다. p2는 data를 1 감소시키고 저장한다. 

p1과 p2가 동시에 일어난다면? p3가 data를 compute하려고 가져왔을 땐 어떤 값이 있어야 하는가?

위와 같은 문제를 race condition이라고 하고  발생시키지 않기 위해서 process synchronization을 하는 것이다.



#### 개념

공유 데이터(shared data)의 동시 접근(concurrent access)은 **데이터의 불일치 문제를 발생**시킬 수 있다.

**일관성(consistency) 유지를 위해서는 협력 프로세스(cooperating process)간의 실행 순서를 정해주는 메커니즘이 필요하다**.



### Race Condition

#### 개념

- 여러 프로세스들이 동시에 공유 데이터를 접근하는 상황
- 데이터의 최종 연산 결과는 접근이 발생한 순서에 의존한다.

=> race condition을 막기 위해서는 concurrent process는 동기화되어야 한다.



#### 발생 경우

1. **Kernel 수행 중 인터럽트 발생 시**

   - kernel mode running 중 interrupt가 발생하여 인터럽트 처리루틴이 수행

     - 양쪽 다 커널 코드이므로 kernel address space 공유

   - 해결책

     - kernel mode에서 진행 중일 때 인터럽트를 막는다.

     

2. **Process가 system call 하여 kernel mode로 수행 중인데 context switch가 일어나는 경우**

   - Process A가 kernel mode로 count값 증가 수행 중에 time expire 하여 Process B에 CPU를 넘겨주었다.

     Process B는 user mode + kernel mode에서 count 값을 증가하게 되고 CPU가 다시 A로 넘어왔다.

     Process A는 수행하던 count 증가를 마저 완료한다. 그러나 결과는 count가 2 증가한다. 이는 중간에 CPU를 선점당했기 때문이다.

   - 해결책
     - kernel mode에서 수행 중일 때는 CPU를 선점하지 않는다.
     - kernel mode에서 user mode로 넘어갈 때 선점한다.

   

3. **Multiprocessor에서 shared memory 내의 kernel data**

   - multiprocessor의 경우 interrupt disable/enable로 해결되지 않는다.

   - 해결책
     - 한 번에 하나의 CPU만이 커널에 들어갈 수 있게 한다. => 비효율적이다.
     - 커널 내부에 있는 각 공유 데이터에 접근할 때마다 그 데이터에 대한 lock/unlock를 한다.





### Critical Section

#### 개념

- N개의 프로세스가 공유 데이터를 동시에 사용하기를 원하는 경우
- 각 프로세스의 code segment에는 공유 데이터를 접근하는 코드인 critical section이 존재
- Problem
  - 하나의 프로세스가 critical section에 있을 때 다른 모든 프로세스는 critical section에 들어갈 수 없어야 한다.



### Critical Section의 해결법

#### 충족조건

- Mutual Exclusion (상호 배제)
  - 프로세스 P1가 critical section 부분을 수행 중이면 다른 모든 프로세스들은 그들의 critical section에 들어가면 안된다.
- Progress (진행)
  - 아무도 critical section에 있지 않은 상태에서 critical section에 들어가고자 하는 프로세스가 있으면 critical section에 들어가게 해주어야 한다.
- Bounded Waiting (한정 대기)
  - 프로세스가 critical section에 들어가려고 요청한 후부터 그 요청이 허용될 때까지 다른 프로세스들이 critical section에 들어가는 횟수에 한계가 있어야 한다.

- 가정
  - 모든 프로세스의 수행 속도는 0보다 크다
  - 프로세스들 간의 상대적인 수행 속도는 가정하지 않는다.



#### 해결 알고리즘

**가정 : 프로세스는 두 개가 있다.**

**Algorithm 1**

turn이라는 변수를 통해 프로세스의 차례를 가리킨다.

```c
int turn;
// Process 0
do{
    while(turn !=0); 	// 본인차례 기다리기, Process 1은 turn !=1
    critical section;
    turn = 1;			// Process 1은 turn = 0 으로 바꿔준다.
    remainder section
}while(1);    
```

문제점은 해당 차례의 프로세스가 critical section을 들어갔다가 나오면 차례가 바뀐다.

그런데 만약 하나의 프로세스는 빈번히 critical section을 들어가고, 다른 하나의 프로세스는 critical section을 한 번만 들어갔다가 나온다면 

**빈번히 들어가야 하는 프로세스는 더 이상 들어가지 못한다.** 

> 순서를 자세히 들여다 보자
>
> 자주 들어가는 프로세스를 p0, 한 번 들어가는 프로세스를 p1이라 하자.
>
> 처음에 turn이 0이어서 p0은 critical section을 들어갔다. 그리고 나와서 turn을 1로 바꾸고 remainder section을 진행한다.
>
> 이젠 p1 차례다. p1도 똑같이 들어갔다가 나오고 turn을 0으로 바꿔준다. 
>
> 다시 p0의 차례다. p0은 들어갔다가 turn을 1로 바꿔준다. 
>
> 근데 p1은 더이상 임계구역에 안 들어갈 거고 이미 끝났을 것이다. 그러면 p0은? 계속 기다리기만 할 것이다.

이는 Progress 조건을 만족시키지 못한다.

<br/>

**Algorithm 2**

flag를 사용한다.

```c
bool flag[2]; // flag는 Process가 critical section에 들어갈 수 있는 지 여부
			  //flag[i] = false, no one in critical section

//Process i
do{
    flag[i] = true;		// I am in!
    while(flag[j]);		// Is he also in? then wait
    critical section;	
    flag[i] = false;	// I am out now!
    remainder section;
}while(1);
```

이는 flag를 이용해서 상대방이 들어가려고 하는지를 확인하고 들어간다. 

당연히 맞는 거 아닐까 싶지만 역시 문제가 있다.

만약 process i가 flag[i] = true까지 실행하고 CPU를 뺏기면 어떻게 될까?

process j가 cpu를 받아서 flag[j] = true를 실행할 것이다. 그리고 무한 대기를 할 것이다. 왜냐? flag[i]도 true이기 때문이다. 

즉, 서로 양보만 하다가 아무 일도 못하는 것이다.

<br/>

**Algorithm 3 (Peterson's Algorithm)**

이제 위의 알고리즘 두 개를 보완한 것이 Peterson's Algorithm이다.

  알고리즘은 flag, turn 변수를 모두 사용한다.

```c
int turn;
bool flag[2];

//Process i
do{
    flag[i] = true;		// i am ready to enter
    turn = j;			// set to his turn
    while(flag[j] && turn ==j);	// wait
    critical section;
    flag[i] = false;
    remainder section;
}while(1);
```

flag로 내가 들어가겠다고 하고 turn을 상대방에게 넘겨준다. 

그리고 while문을 통해 대기하는데, 만약 상대방이 flag가 false라면 critical section에 들어가지 않겠다는 것이므로 내가 들어가면 된다.

또한, turn이 내 차례면 내가 들어가면 된다. 

Algorithm 2에서는 CPU를 뺏기면 문제가 됐지만 여기서는 문제 되지 않는다. (과정은 각자 생각해보길)

<br/>

이렇게 되면 critical 해결법의 3조건을 모두 충족한다!

다만 문제점이 있다. 

- Busy Waiting = spinlock
  - 만약 프로세스가 critical section에 들어갔다면 다른 프로세스는 while문을 돌면서 대기할 것이다. 
  - 그런데 while문을 얼만큼 도느냐? CPU 할당시간만큼 돌 것이다. 즉, CPU 할당 시간만큼 명령어를 실행하니 낭비하는 것이다.

<br/>

#### Hardware TestAndSet

위의 알고리즘은 software적으로 해결한 것이다. 

사실 HW적으로 하나의 실행이 가능하면 쉽게 해결할 수 있다.

문제가 생기는 이유는 

1. 차례를 확인하고
2. 차례를 변경해준다.

와 같은 두 개의 명령어를 실행해주고 도중에 CPU를 선점당할 수 있기 때문이다. 

두 개의 명령어를 한 번에 실행이 가능하다면 쉽게 critical section문제를 해결할 수 있다.

그것이 바로 Test_and_Set()이다.

```c
boolean lock = false;

//Process i

```





### Semaphores

일종의 추상 자료형

**Semaphore S**

- integer variable
- 아래의 2가지 **atomic 연산**에 의해서만 접근 가능하다.
  - P(S) : 자원을 가져가는 것 (= lock = wait)
  - V(S) : 자원을 갖다놓는 것 (= unlock = signal)

```c
P(S){ 	// = wait(S)
    while(S<=0); // busy-wait
    S--;
}
V(S){	// = signal(S)
    S++;		// release
}
```

**중요한 것은 위 연산은 atomic 해야 한다. 도중에 인터럽트되면 안된다.**

<br/>

#### Two Types of Semaphore

세마포어는 두 가지의 타입을 가진다.

- Counting semaphore
  - 도메인이 0 이상인 임의의 정수값
  - 유한한 개수를 가진 자원에 접근을 제어하는데 사용된다.
  - 주로 resource counting에 사용
- Binary semaphore( = mutex)
  - 0 또는 1 값만 가질 수 있는 semaphore
  - 주로 mutual exclusion(lock/unlock)에 사용

<br/>

#### Critical Section of N Processes

semaphore를 이용한 코드를 보자.

```c
//Synchronization variable
sempaphore mutex;

//Process i;
do{
    P(mutex);	// if positive, decrease & enter, otherwise, wait;
    critical section;
    V(mutext);	// Increment semaphore = unlock
    remainder section;
}while(1);
```

그러나 P() 연산이 누군가 critical section에 있다면 계속 while문을 돌면서 기다리므로 busy-wait 문제는 발생한다 = (spin lock)

따라서 busy-wait문제를 **Block & Wakeup 방식의 구현으로 해결한다. ( = sleep lock)**

<br/>

#### Block/Wakeup Implementation

Process는 blocked라는 상태가 있다. 이를 이용하여 바쁜 대기 대신 일시 중지시키는 것이다.

일시 중지 연산은 프로세스를 세마포에 연관된 대기 큐에 넣는다.

Semaphore를 다음과 같이 정의한다. 

```c
typedef struct{
    int value;			// semaphore
    struct process *L;	//process wait queue
}sempaphore;
```



block과 wakeup를 다음과 같이 가정한다.

- Block 
  - 커널은 block를 호출한 프로세스를 suspend시킴
  - 이 프로세스의 PCB를 semaphore에 대한 wait queue에 넣음
- Wakeup(P)
  - block된 프로세스 P를 wakeup시킴
  - 이 프로세스의 PCB를 ready queue로 옮김



Semaphore 연산이 이제 다음과 같이 정의된다.

```c
P(S){
    S.value--;		// prepare to enter
	if(S.value<0){	// if negative, 사용할 수 있는 자원이 없으니 대기해야 한다.
    	add this process to S.L;	// wait queue에 넣고 block
	    block();
	}
}

V(S){
    S.value++;
	if(S.value<=0){	// 누군가 기다리고 있다!
	    remove a process P from S.L;
    	wakeup(P);
	}
}
```

<br/>

#### Busy-wait vs Block/Wakeup

Critical Section 길이 vs Block/Wakeup overhead

- critical section의 길이가 긴 경우 Block/Wakeup이 적당
- critical section의 길이가 짧은 경우 Block/wakeup  overhead가 busy-wait overhead보다 더 커질 수 있음
- 일반적으로는 Block/Wakeup 방식이 더 좋다.

<br/>

### Monitor

모니터란 semaphore의 문제점을 처리하기 위한 동기화 도구이다. 

- Semaphore의 문제점
  - 코딩하기 어렵다
  - 정확성의 입증이 어렵다
  - 자발적 협력이 필요하다
  - 한번의 실수가 모든 시스템에 치명적 영향
    - EX.  V(S) -> critical section -> P(S) 순으로 작성된 경우.
  - EX2. V(S) -> critical section -> V(S) 순으로 작성된 경우. 
  
- 동시 수행중인 프로세스 사이에서 abstract data type의 안전한 공유를 보장하기 위한 **high-level synchronization construct**

  = 모니터 안에서만 shared data을 접근할 수 있다. 

  ```c
  monitor monitor-name{
      //shared variable declarations
      function P1(...){
          ...
      }
      function P2(...){
          ...
      }
      function Pn(...){
          ...
      }
      {
          initialization code
      }
  }
  ```

-  monitor내에서만 진행되므로 lock을 걸 필요 없다.
- 모니터 내에서는 한 번에 하나의 프로세스만이 활동 가능하다.
- 프로그래머가 동기화 제약 조건을 명시적으로 코딩할 필요 없다.
- 프로세스가 모니터 안에서 기다릴 수 있도록 하기 위해 conditional variable를 사용한다.
- Conditional variable은 wait와 signal 연산에 의해서만 접근 가능하다.
  - x.wait()을 invoke한 프로세스는 다른 프로세스가 x.signal()을 invoke하기 전까지 suspend된다.
  - x.signal()은 정확하게 하나의 suspend된 프로세스를 resume한다. suspend된 프로세스가 없으면 아무 일도 일어나지 않는다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week3_ch6_img1.jpg?raw=true" alt="week3_ch6_img1.jpg" style="zoom:50%;" />

```c
//Monitor을 활용한 Producer-Consumer Problem
monitor bounded_buffer
{
    int buffer[N];
    condition full,empty;
    // condition var은 값을 가지지 않고 자신의 큐에 프로세스를 매달아서 sleep시키거나 큐에서 프로세스를 깨우는 역할만 한다.
    
    void produce(int x){
        if there is no empty buffer
            empty.wait();
        add x to an empty buffer;
        full.signal();
    }
    
    void consume(int *x){
        if there is no full buffer
            full.wait();
        remove an item from buffer and store it to *x;
        empty.signal();
    }
}
```

<br/>

### Deadlock and Starvation

#### Deadlock

둘 이상의 프로세스가 서로 상대방에 의해 충족될 수 있는 event를 무한히 기다리는 현상

$$
P_0 \qquad  P_1 \\
wait(S); \quad wait(Q);\\
wait(Q); \quad wait(S);\\
\cdot \qquad\qquad \cdot\\
\cdot \qquad\qquad \cdot\\
\cdot \qquad\qquad \cdot\\
signal(S); \quad signal(Q);\\
signal(Q); \quad signal(S);\\
$$
위와 같이 P0이 wait(S)를 실행하고 P1이 wait(Q)를 실행한다고 가정하자.

그렇다면 P0은 wait(Q)를 실행할 때, P1이 signal(Q)를 할 때까지 기다려야 한다. 그렇다면 P1이 wait(S)하는 경우도 동일하다.

즉, 자원을 갖고 있고 다른 자원을 요구하지만 서로 갖고 있는 것이어서 진행되지 않고 무한히 대기하게 된다.

<br/>

#### Starvation

- Indefinite blocking : 프로세스가 suspend된 이유에 해당하는 semaphore queue에서 빠져나갈 수 없는 현상

<br/>

