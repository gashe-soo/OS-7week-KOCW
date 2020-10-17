# CH 4. Process Management

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.



### 프로세스 생성(process creation)

부모 프로세스가 자식 프로세스를 생성 = 복제 생성 = 부모의 context 복제해서 자식 생성

=> OS에 따라 다르지만 리눅스의 경우 일단 주소 공유하다가 프로세스가 달라지면 복제하여 따로 진행 = Copy-on write(COW) = Write가 발생했을 때 copy한다.

프로세스의 트리(계층 구조)를 형성한다.

프로세스는 자원을 필요로 한다.

- 운영체제로부터 받는다.
- 부모와 공유

자원의 공유

- 부모&자식이 모든 자원 공유
- 일부 공유
- 전혀 공유X => 일반적, 서로 자원을 차지하기 위해 경쟁

수행

- 부모와 자식 공존하며 수행
- 자식이 종료될 때까지 부모가 기다리는 모델

주소 공간

- 자식은 부모의 공간을 복사함 (binary and OS data)
- 자식은 그 공간에 새로운 프로그램을 올림

유닉스의 예

- fork() 시스템 콜이 새로운 프로세스를 생성
  - 부모를 그대로 복사
  - 주소 공간 할당
- fork 다음에 이어지는 exec() 시스템 콜을 통해 새로운 프로그램을 메모리에 올림 ( 자식이 없더라도 exec를 통해서 새로운 프로그램 메모리에 올리기 가능)



### 프로세스 종료

프로세스가 마지막 명령을 수행한 후 OS에게 이를 알려줌(exit) 

- 자식이 부모에게 output data를 보냄(via wait) => 자식이 부모보다 먼저 terminate
- 프로세스의 각종 자원들이 OS에게 반납

부모 프로세스가 자식의 수행을 종료시킴 (abort)

- 자식이 할당 자원의 한계치를 넘어섬
- 자식에게 할당된 태스크가 더 이상 필요하지 않음
- 부모가 종료하는 경우
  - OS는 부모 프로세스가 종료하는 경우 자식이 더 이상 수행되도록 두지 않는다.
  - 단계적인 종료



### 프로세스 관련 시스템 콜

#### fork

프로세스는 fork 시스템 콜에 의해 생성된다.

```c
int main(){
    int pid;
    pid = fork();
    if(pid == 0) /*this is child*/
        printf("Hello, I am child\n");
    else if(pid > 0) /*this is parent*/
        printf("Hello, I am parent!\n");
}
```

fork시작하면 cpu 수행 단계도 복제하므로 자식 프로세스는 if문부터 시작한다.

<br/>

#### exec

exec() system call은 다른 프로그램을 실행시킨다.  

exec 이후에 존재하던 코드를 실행하지 않는다. 다른 프로그램을 실행하기 때문

<br/>

#### wait

process A가 wait() 시스템 콜을 호출하면

- 커널은 child가 종료될 때까지 프로세스 A를 sleep (block)
- Child process가 종료되면, 커널은 프로세스 A를 깨운다 (ready)

> 좀비 프로세스 (zombie process)
>
> 프로세스의 종료 상태가 저장되는 프로세스 테이블의 항목은 부모가 wait() 호출할 때까지 남아있게 된다.
>
> **종료되었지만 부모 프로세스가 아직 wait() 호출을 하지 않은 프로세스를 '좀비 프로세스'라고 한다.**

> 고아 프로세스(orphan process)
>
> 부모 프로세스가 wait를 호출하는 대신 종료한다면? => 갈 길을 잃는다 
>
> 이것을 고아 프로세스라 한다.

<br/>

#### exit

프로세스의 종료

- 자발적 종료
  - 마지막 statement 수행 후 exit 시스템 콜을 통해
  - 프로그램에 명시적으로 적어주지 않아도 main함수가 리턴되는 위치에 컴파일러가 넣어준다.
- 비자발적 종료
  - 부모 프로세스가 자식 프로세스를 강제로 종료
    - 자식 프로세스가 한계치 넘어서는 자원 요청
    - 자식에게 할당된 태스크가 더 이상 필요하지 않음
  - kill, break 등
  - 부모가 종료하는 경우
    - 부모 프로세스가 종료하기 전에 자식들이 먼저 종료됨

<br/><br/>

### 프로세스간 협력

#### 독립적 프로세스(Independent process)

- 프로세스는 각자의 주소 공간을 가지고 수행되므로 원칙적으로 하나의 프로세스는 다른 프로세스의 수행에 영향을 미치지 못함

<br/>

#### 협력 프로세스(Cooperating process)

- 프로세스 협력 메커니즘을 통해 하나의 프로세스가 다른 프로세스의 수행에 영향을 미칠 수 있음
- 프로세스간 협력을 허용하는 환경을 제공하는 이유
  - 정보 공유
  - 계산 가속화 : 특정 태스크를 서브 태스크로 나누어 병렬 실행
  - 모듈성 : 시스템 기능을 별도의 프로세스/스레드로 나누어 실행

<br/>

#### 프로세스 간 협력 메커니즘(IPC : InterProcess Communication)

<img src="https://blog.kakaocdn.net/dn/blJvFo/btqDurr5tfi/2TlZ1kwuqxNyCtZ1hdTyUK/img.png" alt="OS" style="zoom:60%;" />

출처 : [XXXX - OS IPC](https://xxxxxxxxxxxxxxxxx.tistory.com/entry/OS-Inter-Process-Communication-IPC)

- 메세지를 전달하는 방법
  - message passing : 커널을 통해 메시지 전달
  - 충돌 회피할 필요가 없기 때문에 적은 양의 데이터 교환에 유용
- 주소 공간을 공유하는 방법
  - shared memory : 서로 다른 프로세스 간에도 일부 주소 공간을 공유하게 하는 shared memory 메커니즘이 있음 => 공유 메모리 영역을 구축할 때만 시스템 콜 필요.
  - message passing보다 빠른 메시지 전달.
  - thread : thread는 사실상 하나의 프로세스이므로 프로세스 간 협력으로 보기는 어렵지만 동일한 process를 구성하는 thread들 간에는 주소 공간을 공유하므로 협력이 가능



#### message passing

message system : 프로세스 사이에 공유 변수(shared variable)를 일체 사용하지 않고 통신하는 시스템

1. Direct Communication
   - 통신하려는 process의 이름을 명시적으로 표시
   - P: Send(Q, message) => Q: Receive (P, message)
2. Indirect Communication
   - mailbox ( or port)를 통해 메시지를 간접 전달
   - P : Send(M, message) => Mailbox(M) => Q : Receive(M,message)



### REFERENCE

[XXXX - OS IPC](https://xxxxxxxxxxxxxxxxx.tistory.com/entry/OS-Inter-Process-Communication-IPC)