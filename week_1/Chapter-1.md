# CH 1. INTRODUCTION TO OS

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.



### 운영체제란?

컴퓨터 HW 바로 위에 설치되어 사용자 및 다른 모든 SW와 HW를 연결하는 SW 계층

- 협의 : 운영체제의 핵심 부분으로 메모리에 상주하는 부분
- 광의 : 커널 뿐 아니라 각종 주변 시스템 유틸리티를 포함한 개념

![운영 체제 - 위키백과, 우리 모두의 백과사전](https://upload.wikimedia.org/wikipedia/commons/a/a3/Operating_system_placement_kor.png)





### 목적

1. 사용자가 컴퓨터 시스템을 편리하게 사용할 수 있는 환경 제공

2. 컴퓨터 시스템의 자원을 효율적으로 관리 = 제어 프로그램
   - 프로세서, 기억장치, 입출력 장치를 관리
     - 사용자간의 형평성 있는 resource 분배
     - 한정된 resource로 최대한의 성능을 내도록
   - 사용자 및 운영체제 자신의 보호
   - 프로세스, 파일, 메세지 등을 관리



### 분류

1. 동시 작업 가능 여부
   - 단일 작업(Single Tasking) : 한 번에 하나의 작업만 처리
   - 다중 작업(Multi Tasking) : 동시에 두 개 이상의 작업 처리
2. 사용자의 수
   - 단일 사용자 ex. MS windows
   - 다중 사용자 
     - 다중 사용자가 동시 접근, 형평성을 고려해야 한다. 
     - ex. UNIX
3. 처리 방식
   - 일괄 처리(batch processing)
     - 작업 요청의 일정량 모아서 한꺼번에 처리
   - 시분할(time sharing)
     - 여러 작업을 수행할 때 일정 시간 단위로 분할하여 CPU 사용
     - 일괄 처리에 비해 짧은 응답 시간을 가짐
     - Interactive한 방식
   - 실시간 (Realtime)
     - 정해진 시간 안에 어떠한 일이 반드시 종료됨이 보장되어야 한다.
     - Hard Realtime System ex. 미사일 제어, 반도체 장비 
     - Soft Realtime System  ex. 스트리밍



> 참고
>
> MultiTasking : 여러 작업이 동시에 수행되는 것
>
> MultiProgramming : 여러 프로그램이 메모리에 올라가 있음을 강조 => 메모리 강조
>
> Time Sharing : CPU의 시간 분할하여 나누어 쓴다 => CPU 강조
>
> Multiprocess : 여러 프로그램이 동시에 실행
>
> 하나의 CPU
>
> -----
>
> MultiProcessor
>
> - CPU가 여러 개 있는 컴퓨터.



### 종류

#### UNIX

- 높은 이식성 (∵ C언어로 작성)
- 최소한 커널 구조
- 다양한 버전
  - Linux etc

#### MS Windows 

- PC용 소프트웨어
- 불안정성
- 풍부한 지원 소프트웨어





### REFERENCE

[위키백과 - 운영체제]([https://ko.wikipedia.org/wiki/%EC%9A%B4%EC%98%81_%EC%B2%B4%EC%A0%9C](https://ko.wikipedia.org/wiki/운영_체제))