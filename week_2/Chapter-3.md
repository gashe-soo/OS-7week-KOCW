# CH 3. Process

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.



### 개념

프로세스는 실행중인 프로그램이다.



> Process vs Program
>
> Program : 명령어 리스트를 내용으로 가진 디스크에 저장된 파일 ( = 실행 파일)
>
> Process : 다음에 실행할 명령어를 지정하는 program counter와 관련 자원의 집합을 가짐
>
> 실행 파일이 메모리에 적재될 때 프로그램은 프로세스가 된다.

<br/>

### 프로세스의 문맥 (context) 

프로세스의 현재 상태 (프로세스가 어디까지 실행)를 나타낸다.  

- CPU 수행 상태를 나타내는 하드웨어 문맥 

  - Program Counter
  - 각종 register

- 프로세스의 주소 공간

  - code, data, stack

- 프로세스 관련 커널 자료 구조 : 운영체제가 프로세스를 관리하므로 프로세스별 형태를 알고 있다.

  - PCB (Process Control Block)
  - Kernel stack : 프로세스가 하지 못할 경우 OS에게 요청=> 프로세스별로 커널 스택을 따로 둔다.

  <br/>

그렇다면 문맥이 왜 중요한가?

여러 프로세스가 돌아가는데 백업해두지 않으면 CPU가 다시 잡았을 때 어디부터 실행해야 할지를 알고 있기 때문!

<br/>

### 상태

프로세스는 상태(state)가 변경되며 수행된다.



![img](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter3/3_02_ProcessState.jpg)

출처 : [cs.uic.edu](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/3_Processes.html)

- Running :
  - CPU를 잡고 명령어를 수행 중인 상태
- Ready : CPU를 기다리는 상태 => 당장 수행할 수 있는 조건(메모리 등)을 모두 만족해야 한다.
- Blocked(wait, sleep) : 
  - CPU를 주어도 당장 instruction을 수행할 수 없는 상태
  - Process 자신이 요청한 event(I/O)가 즉시 만족되지 않아 이를 기다리는 상태
  - EX. 디스크에서 file을 읽어와야 하는 경우
- New : 프로세스가 생성 중인 상태
- Terminated : 수행이 끝난 상태
- Suspended(stopped) : Medium-term Scheduler로 인해 발생.
  - 외부적인 이유로 프로세스의 수행이 정지된 상태
  - 프로세스는 통째로 디스크에 swap out된다.
  - EX. 사용자가 프로그램을 일시 정지시킨 경우 / 시스템이 여러 이유로 프로세스를 잠시 중단시킴.

![프로세스 상태 전이도 ( Process State Diagram )](https://img1.daumcdn.net/thumb/R720x0.q80/?scode=mtistory2&fname=http%3A%2F%2Fcfile27.uf.tistory.com%2Fimage%2F233F6A3659310C9A37BACA)

출처 : https://junsday.tistory.com/25

>Blocked : 자신이 요청한 event가 만족되면 Ready
>
>Suspended : 외부에서 resume해 주어야 Active

CPU , I/O device는 Queue를 가진다. 각 프로세스는 필요시 해당 큐에 들어가서 대기 => 순서대로 처리

 <br/>

### PCB(Process Control Block)

![img](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter3/3_03_PCB.jpg)

출처 : [cs.uic.edu](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/3_Processes.html)

- OS가 각 프로세스를 관리하기 위해 프로세스당 유지하는 정보
- 구성요소
  - OS가 관리상 사용하는 정보
    - Process state, Process ID
    - scheduling info, priority
  - CPU 수행 관련 하드웨어 값
    - Program Counter, registers
  - 메모리 관련
    - code, data, stack의 위치 정보
  - 파일 관련
    - open file descriptors



<br/>

### 문맥 교환(Context Switch)

- CPU를 한 프로세스에서 다른 프로세스로 넘겨주는 과정
- CPU가 다른 프로세스에게 넘어갈 때 운영체제는 다음을 수행
  - CPU를 내어주는 프로세스의 상태를 그 프로세스의 PCB에 저장
  - CPU를 새롭게 얻는 프로세스의 상태를 PCB에서 읽어옴

![img](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter3/3_04_ProcessSwitch.jpg)



<br/>

> System call이나 Interrupt 발생시 반드시 Context Switch가 일어나는 것은 아니다.
>
> 1) Process A -> (Interrupt / system call) -> Kernel -> Process A 
>
> => 문맥 교환 없음
>
> 2) Process A  -> (timer interrupt or I/O system call)  -> Kernel  -> Process B
>
> => 문맥 교환 일어남
>
> timer = 다른 프로세스에게 넘기기 위함 , I/O는 시간이 오래걸리므로 넘긴다.
>
> 1)의 경우에도 user mode에서 kernel mode로 넘어가므로 CPU 수행 정보 등 context의 일부를 PCB에 저장해야 하지만,
>
> 문맥 교환하는 2)의 경우가 오버헤드 훨씬 큼.

<br/>

### 프로세스 스케줄링 큐



![img](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter3/3_06_QueueingDiagram.jpg)

- Job queue
  - 현재 시스템 내에 있는 모든 프로세스의 집합
- Ready queue
  - 현재 메모리 내에 있으면서 CPU를 잡아서 실행되기를 기다리는 프로세스의 집합
  - 일반적으로 연결 리스트로 저장된다. = 각 PCB에는 준비 큐의 다음 PCB를 가리키는 포인터 필드가 포함된다.
- Device queue
  - I/O device의 처리를 기다리는 프로세스의 집합
- 프로세스들은 각 큐를 오가며 수행된다.



<br/>

### 스케줄러(Scheduler)

- Short-term scheduler( CPU scheduler)
  - 어떤 프로세스를 다음번에 running 시킬지 결정
  - 프로세스에 CPU를 주는 문제
  - 충분히 빨라야 함(ms 단위)

- Long-term Scheduler( Job scheduler)

  - 시작(new) 프로세스 중 어떤 것들을 ready queue로 보낼지 결정

  - 프로세스에 memory를 주는 문제

  - degree of Multiprogramming을 제어 => memory에 올라가는 프로세스의 수를 결정

  - time sharing system에는 보통 장기 스케줄러가 없음(무조건 ready)  

    => 그러면 어떻게 degree of Multiprogramming을 제어하는가? =  medium-term scheduler

- Medium-term scheduler( Swapper)

  - 여유 공간 마련을 위해 일부 프로세스를 통째로 메모리에서 디스크로 쫓아냄 = 메모리에서의 프로세수 수 제어.
  - 프로세스에게서 memory를 뺏는 문제
  - degree of Multiprogramming을 제어 
  - 프로세스의 상태인 Suspended을 발생시킨다.



<br/>

### Thread

- 프로세스 내부의 CPU 수행 단위.

- 구성 = CPU 관련 정보만 별도로 가지고 있다.
  - program counter
  - register set
  - stack space
- Thread가 다른 Thread와 공유하는 부분
  - code, data
  - OS resources

<br/>

장점

- 빠른 응답성
  - 다중 스레드로 구성된 task 구조에서는 하나의 서버 스레드가 blocked(waiting) 상태인 동안에도 동일한 task내의 다른 스레드가 실행되어 빠른 처리 가능
  - EX. multi-threaded web
- 경제성 
  - 프로세스 생성, 문맥 교환보다는 스레드 생성, 문맥 교환이 오버헤드가 적다.
- 성능
  - 동일한 일을 수행하는 다중 스레드가 협력하여 높은 처리율(throughput)과 성능 향상을 얻을 수 있다. 
  - 스레드를 사용하면 병렬성을 높일 수 있다. ( CPU가 여러 개일 경우)

<br/><br/>

### REFERENCE

[cs.uic.edu](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/3_Processes.html)

[Wikipedia - Process State](https://en.wikipedia.org/wiki/Process_state)

[프로그래밍에 지름길은 없다 - 프로세스 상태 전이도](https://junsday.tistory.com/25)

