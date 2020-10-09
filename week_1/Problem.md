# Problem



### Week 1.

1. 다음은 Interrupt-driven I/O Cycle의 과정이다. 박스 안에 정답을 채워 넣으시오.

   ![week1_problem_1.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week1_problem_1.jpg?raw=true)

   - a.  interrupt handler process data, returns from interrupt
   - b. initiates I/O
   - c. CPU receiving interrupt, transfers control to interrupt handler
   - d. CPU resumes processing of interrupted task
   - e. input ready, output complete, or error generates interrupt signal
   - f. device driver initiates I/O

   



2. 다음 중 옳은 것을 모두 고르시오.
   - ㄱ. 운영체제는 사용의 용이성을 위해 설계되었고 자원의 이용에는 신경쓰지 않는다.
   - ㄴ. CPU는 하나의 명령어의 실행을 완료할 때마다 인터럽트 라인을 감지한다.
   - ㄷ. Mode bit이 1이 되면서 커널 모드에 진입한다.
   - ㄹ. 메인 메모리는 비휘발성 저장장치이다.
   - ㅁ. DMA는 CPU의 개입 없이 메모리와 버퍼 장치 간의 데이터를 바이트 단위로 전송한다.
   - ㅂ. Swap area는 메모리에 속한다.
   - ㅅ. Memory Mapped I/O는 메모리와 I/O가 하나의 연속된 address 영역에 할당된다.

-------







정답.

1번 

![week1_problem_1_answer.jpg](https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week1_problem_1_answer.jpg?raw=true)

2번. ㄴ,ㅅ

- ㄱ. 운영체제는 사용의 용이성을 위해 설계되었고 자원의 이용에도 **신경 쓴다.**
- ㄷ. **Mode bit이 0**이 되면서 커널 모드에 진입한다.
- ㄹ. 메인 메모리는 **휘발성** 저장장치이다.
- ㅁ. DMA는 CPU의 개입 없이 메모리와 버퍼 장치 간의 데이터를 **블록** 단위로 전송한다.
- ㅂ. Swap area는 **디스크**에 속한다.