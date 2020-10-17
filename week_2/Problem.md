# Week 2 Problem

1. 다음 코드에서 출력되는 값과 그 이유를 설명하시오.

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <unistd.h>

int value = 5;
int main()
{
    pid_t pid;
    pid = fork();
    
    if(pid == 0){
        value += 15;
        return 0;
    }
    else if(pid > 0){
        wait(NULL);
        printf("Parent: value = %d\n",value);
        return 0;
    }
}
```

출처 : 운영체제(Operating System Concepts) 10th edition



2.  O/X를 고르시오.

   -  같은 프로세스에 있는 각각의 쓰레드는 각각 가상 메모리 mapping을 가진다. (O/X)

   - 쓰레드는 프로세스보다 문맥 교환(context switch) 비용이 적다.(O/X)

   - 한 프로세스 내의 스레드들은 Code, Data, Heap 영역을 공유한다.(O/X)

   - 프로세스가 fork() 연산을 사용하여 새로운 프로세스를 생성할 때, 부모 프로세스와 자식 프로세스는 스택, 힙의 상태를 공유한다. (O/X)





정답

1. 출력된 값은 5이며, 부모 프로세스의 경우 값을 변경하지 않기 때문이다.

2. 해설

   1) X => 스레드는 메모리를 공유한다. [참고](https://stackoverflow.com/questions/5440128/thread-context-switch-vs-process-context-switch)

   2) O => 프로세스의 문맥 교환은 code,data,heap영역도 바뀌어야 하기 때문

   3) O => 스레드는 stack space, program counter만 따로 갖는다.

   4)  X => 부모 프로세스와 자식 프로세스는 공유 메모리 세그먼트만 공유한다.

   