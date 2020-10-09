# CH2. System Structure & Program Execution

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.



### Computer System Structure

컴퓨터는 크게 Computer, I/O Device로 나뉜다.

- Computer
  - CPU 
  - Memory : CPU의 작업공간

- I/O device
  - Disk
  - keyboard, mouse, printer, monitor etc
  
  - device controller: device를 전담하는 cpu 역할 + 작업공간 : local buffer

![week_1_ch2_img1.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week_1_ch2_img1.jpg?raw=true)



#### CPU & Memory

Memory는 프로그램이 실행되는 동안 명령어와 데이터를 저장한다.

CPU는 메모리에서 명령어를 읽어서 실행한다.

CPU는 계속 명령어를 실행해야 하므로 시간이 오래 걸리는 I/O를 기다려줄 여유가 없다. 그래서 cpu가 I/O할 때는 controller에 요청한 후 놀지 않고 다른 명령어를 실행한다.  

그렇다면 I/O가 끝났을 때는? CPU에게 I/O가 끝났다고 알려주어야 한다. 이것이 Interrupt이다.



#### I/O Device Controller

- 해당 I/O 장치 유형을 관리하는 일종의 작은 CPU
- CPU와 병렬로 실행되어 메모리 사이클을 놓고 경쟁한다. (누가 메모리에 접근할 것인가)
- 제어 정보를 위해 control register, status register을 가짐
- local buffer를 가짐
- I/O는 실제 device와 local buffer 사이에서 일어남
- I/O가 끝났을 경우 인터럽트로 CPU에 사실 알림



> device driver(장치 구동기) : OS 코드 중 각 장치별 처리루틴 -> SW
>
> device controller(장치 제어기) : 각 장치를 통제하는 일종의 작은 CPU -> HW

#### Mode bit

- 사용자인지 OS인지 모드를 나타나기 위한 비트
- 사용자 프로그램의 잘못된 수행으로 다른 프로그램 및 운영체제에 피해가 가지 않도록 하기 위해 보호 장치 필요
  - 사용자모드 ( 1 ) : 사용자 프로그램 수행
  - 커널 모드 ( 0 ) :  OS 코드 수행
    - 모든 명령 수행 가능 ( = 특권 명령)

- 보안을 해칠 수 있는 중요한 명령어는 커널 모드에서만 수행가능한 특권 명령으로 규정

- 인터럽트나 Exception 발생시 하드웨어가 mode bit을 0으로 바꿈

- 보안 상의 문제로 사용자모드에서는 명령 수행 제한

<br/>

#### Timer

- 정해진 시간이 흐른 뒤 운영체제에게 제어권이 넘어가도록 인터럽트 발생
- 매 클럭 틱 때마다 1감소
- 타이머 0이 되면 타이머 인터럽트 발생
- CPU를 특정 프로그램이 독점하는 것으로부터 보호
- Time Sharing 구현하기 위해 이용됨

<br/>

### I/O & Interrupt

#### I/O 수행

모든 입출력 명령은 커널 모드에서만 할 수 있다. (= 특권 명령)

그렇다면 사용자 프로그램은 어떻게 I/O하는가?

1.  사용자프로그램이 I/O를 할 수 없으므로 OS에게 System Call을 요청한다.

   - System call : 사용자프로그램이 OS에게 I/O 요청하는 함수이다.
     - 단순히 메모리 주소 점프가 아님
     - 인터럽트를 걸도록 한다.

   - 올바른 I/O 요청인지 확인 후 I/O 수행

2. System Call을 요청하면 Mode bit이 0으로 바뀌면서 제어권이 OS에게 넘어가고 I/O을 수행한다. 
3. I/O을 할 동안에는 CPU가 다른 일을 하고 있으므로 I/O 완료되면 CPU에게 Interrupt를 걸게된다. 

4. Interrupt가 들어오면 CPU는 실행 중인 명령을 완료하고 인터텁트 벡터가 가리키는 인터럽트 서비스 루틴을 실행한다.
5. 인터럽트 서비스 루틴이 끝나면 CPU는 다시 시스템 콜 다음 명령을 실행한다.



인터럽트에 대해 더 알아보자

<br/>

#### 인터럽트

CPU에게 인터럽트가 들어왔다. 

그렇다면 인터럽트 당한 시점의 레지스터와 Program Counter를 save한 후 CPU는 커널의 인터럽트 핸들러로 점프한다.

인터럽트 핸들링이 모두 끝나면 다시 저장했던 PC로 돌아와서 명령어를 실행한다.

**종류**

- Interrupt( HW Interrupt ) :
  - 하드웨어가 발생시킨 인터럽트
- Trap( SW Interrupt ) : 
  - Exception : 프로그램이 오류를 범한 경우
  - System call : 프로그램이 커널 함수를 호출하는 경우

**인터럽트 벡터? 인터럽트 핸들러?**

- 인터럽트 벡터 :
  - 해당 인터럽트의 처리 루틴 주소를 가지고 있음
- 인터럽트 서비스 루틴 ( = 인터럽트 핸들러 )
  - 해당 인터럽트를 처리하는 커널 함수( = 인터럽트의 종류마다 정해진 처리 과정)
- 인터럽트가 없다면 OS가 CPU를 가지고 있을 일은 없다.



> 참고
>
> Interrupt request line?
>
> 하나의 명령어의 실행을 완료할 때마다 CPU는 Interrupt request line을 감지한다.
>
> Device Controller가 라인에 신호를 보낸 것을 감지하면 인터럽트 번호를 읽고 인터럽트 핸들러 루틴으로 점프한다.
>
> 이후 처리한다.



그런데 만약 I/O에서 대량이나 고속으로 데이터 이동이 된다면? 

오버헤드가 일어나거나 빈번히 CPU에게 인터럽트를 발생시키고 사용해야 할 것이다. 이는 비효율적이다.

따라서 이를 해결하기 위해 DMA를 사용한다.

<br/>

#### DMA(Direct Memory Access)

- 빠른 I/O 장치를 메모리에 가까운 속도로 처리하기 위해 사용

- CPU의 중재없이 device controller가 device의 buffer storage의 내용을 메모리에 block 단위로 직접 전송
- 바이트 단위가 아니라 **block 단위**로 인터럽트 발생시킴

=> CPU는 DMA가 일을 다 하면 한번만 인터럽트 당함.

<br/>

#### 동기식 I/O 

- I/O 요청 후 입출력작업이 완료된 후에야 제어가 사용자 프로그램에 넘어감
- 방법 1
  - I/O 끝날 때까지 CPU를 낭비시킴
  - 매 시점 하나의 I/O만 가능
- 방법 2
  - I/O 완료될 때까지 해당 프로그램에게서 CPU 빼앗음
  - 다른 프로그램에게 CPU 할당

<br/>

#### 비동기식 I/O

- I/O 시작된 후 입출력 작업이 끝나기를 기다리지 않고 제어가 사용자 프로그램에 즉시 넘어감.



두 경우 모두 I/O가 완료되면 Interrupt 발생한다.



#### **I/O 수행 방식**

1. Memory Mapped I/O
   - 메모리와 I/O가 **하나의 연속된 address 영역에 할당**된다
2. I/O-Mapped I/O
   - 메모리와 I/O가 별개의 어드레스 영역에 할당되는 것을 의미한다. 

<br/>



### 저장장치 계층 구조

![week_1_ch2_img2.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week_1_ch2_img2.jpg?raw=true)

1. 휘발성
   - 휘발성 저장장치 : 레지스터 / 캐시 / 메인 메모리
   - 비휘발성 저장장치 : 하드디스크 / 광학 디스크 / 자기 테이프 등
2. 속도 
   - 계층이 올라갈수록 액세스 속도가 빠르다.
3. 용량
   - 계층이 올라갈수록 용량이 작고 비싸다.



### Program Execution

파일시스템의 실행파일이 메모리에 프로세스로 load



1. 파일시스템의 실행파일

2. 가상 메모리의 독자적인 주소 공간 생성
   - code : 커널 코드
     - 시스템콜, 인터럽트 처리 코드
     - 자원 관리 코드, 서비스 제공을 위한 코드
   - data : PCB, CPU, memory, disk
   - stack : kernel stack
3. 물리적인 메모리에 load
   - 공간 낭비를 막기 위해 가상 메모리 전체를 올리지 않고 필요한 부분만 load.
   - 필요하지 않으면 swap area에 저장.  (swap area도 hard disk에 속함)



#### 함수

- 사용자 정의 함수
- 라이브러리 함수
  - 프로그램 실행 파일에 포함
- 커널 함수
  - 운영체제 프로그램 함수
  - 커널 함수 호출 = 시스템 콜





