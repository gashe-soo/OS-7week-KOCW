# CH 11. Disk Management and Scheduling

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.

<br/>

<br/>

### Disk Structure

#### Logical block 

- 디스크의 외부에서 보는 디스크의 단위 정보 저장 공간들
- 주소를 가진 1차원 배열처럼 취급
- 정보를 전송하는 최소 단위

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch11_img1.jpg?raw=true" alt="week7_ch11_img1.jpg"/>

#### Sector

- Logical block이 물리적인 디스크에 매핑된 위치
- Sector 0은 최외곽 실린더의 첫 트랙에 있는 첫 번째 섹터이다.

<br/>

### Disk Management

#### Physical formatting(Low-level formatting)

- 디스크를 controller가 읽고 쓸 수 있도록 sector들로 나누는 과정
- 각 섹터는 header + 실제 data (보통 512 bytes) + trailer로 구성
- header와 trailer는 sector number, ECC(Error-Correcting Code) 등의 정보가 저장되며 controller가 직접 접근 및 운영

<br/>

#### Partitioning

- 디스크를 하나 이상의 실린더 그룹으로 나누는 과정
- OS는 이것을 독립적 disk로 취급(logical disk)

<br/>

#### Logical formatting

- 파일 시스템을 만드는 것
- FAT, inode, free space 등의 구조 포함

<br/>

#### Booting

- ROM에 있는 "small bootstrap loader" 실행
- sector 0 (boot block)을 load하여 실행
- sector 0은 "full Bootstrap loader program"
- OS를 디스크에서 load하여 실행

<br/>

### Disk Scheduling

#### Access time의 구성

- Seek time
  - 헤드를 해당 실린더로 움직이는데  걸리는 시간
- Rotational latency
  - 헤드가 원하는 섹터에 도달하기까지 걸리는 회전지연시간
- Transfer time
  - 실제 데이터의 전송 시간

<br/>

#### Disk Bandwidth

- 단위 시간당 전송된 바이트의 수

<br/>

#### Disk Scheduling

- seek time을 최소화하는 것이 목표
- Seek time 은 seek distance 에 비례

<br/>

### Disk Scheduling Algorithm

큐에 다음과 같은 실린더 위치의 요청이 존재하는 경우 디스크 헤드 53번에서 시작한 각 알고리즘의 수행 결과를 알아보자. (실린더의 위치는 0-199이다)

98, 183, 37, 122, 14, 124, 65, 67

<br/>

#### FCFS (First Come First Served)

흔히 아는 선입 선처리이다. 

공평해보이지만, 빠른 서비스를 제공하지는 못한다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch11_img2.jpg?raw=true" alt="week7_ch11_img2.jpg" style="zoom: 67%;" />

출처 : 강의

<br/>

#### SSTF(Shortest Seek Time First)

현재의 헤드로부터 가까운 거리에 있는 실린더부터 확인하는 알고리즘이다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch11_img3.jpg?raw=true" alt="week7_ch11_img3.jpg"/>

문제점은 가까운 거리에 있는 것부터 확인하므로 starvation 문제가 발생한다.

<br/>

#### SCAN

disk arm이 디스크의 한쪽 끝에서 다른쪽 끝으로 이동하며 가는 길목에 있는 모든 요청을 처리한다.

다른 한쪽 끝에 도달하면 역방향으로 이동하며 오는 길목에 있는 모든 요청을 처리하며 다시 반대쪽 끝으로 이동한다.

그래서 엘리베이터 알고리즘이라고도 불린다.

문제점은 실린더 위치에 따라 대기 시간이 다르다는 것이다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch11_img4.jpg?raw=true" alt="week7_ch11_img4.jpg" style="zoom:150%;" />

<br/>

#### C-SCAN

SCAN의 문제점을 해결하기 위한 알고리즘이다. 

헤드가 한쪽 끝에서 다른쪽 끝으로 이동하며 요청을 처리하는 것은 동일하다. 

다만 다른쪽 끝에 도달했으면 출발점으로 다시 이동하여 요청을 처리한다. 즉, 순환하기 때문에 C(circular)-SCAN알고리즘이다.

따라서 SCAN보다는 균일한 대기 시간을 제공한다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch11_img5.jpg?raw=true" alt="week7_ch11_img5.jpg" style="zoom:150%;" />

<br/>

#### N-SCAN

SCAN의 변형 알고리즘이다. 일단 한 방향으로 움직이기 시작하면 그 시점 이후에 도착한 job은 되돌아올 때 처리한다.

<br/>

#### LOOK & C-LOOK

SCAN이나 C-SCAN은 헤드가 디스크 끝에서 끝으로 이동했다.

LOOK은 이를 보완하여 이동 방향에 더이상 요청이 없으면 이동방향을 바꾸어 반대로 이동한다. 

다음은 C-LOOK이다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch11_img6.jpg?raw=true" alt="week7_ch11_img6.jpg" style="zoom:67%;" />

<br/>

#### Disk-Scheduling Algorithm의 결정

- SCAN, C-SCAN 및 응용 알고리즘은 LOOK, C-LOOK 등이 일반적으로 디스크 입출력이 많은 시스템에서 효율적인 것으로 알려져 있다.
- File의 할당 방법에 따라 디스크 요청이 영향을 받는다.
- 디스크 스케줄링 알고리즘은 필요할 경우 다른 알고리즘으로 쉽게 교체할 수 있도록 OS와 별도의 모듈로 작성되는 것이 바람직하다.

<br/>



### Swap-Space Management

#### Disk를 사용하는 2가지 이유

- memory의 volatile한 특성 -> file system
- 프로그램 실행을 위한 memory 공간 부족 -> swap space

<br/>

#### Swap - Space

- Virtual memory system에서는 디스크를 memory의 연장 공간으로 사용
- 파일 시스템 내부에 둘 수도 있으나 별도 partition 사용이 일반적
  - 공간 효율성보다는 속도 효율성이 우선
  - 일반 파일보다 훨씬 짧은 시간만 존재하고 자주 참조된다.
  - 따라서, block의 크기 및 저장 방식이 일반 파일시스템과 다르다.



<br/>



### RAID(Redundant Array of Independent Disks)

RAID는 여러 개의 디스크를 묶어서 사용하는 것이다.

그렇다면 왜 그렇게 사용할까?

RAID의 사용 목적은 2가지이다.

1. 디스크 처리 속도 향상
   - 여러 디스크에 block의 내용을 분산 저장하여 병렬적으로 읽어오면서(Interleaving, striping) 속도를 향상시킨다.
2. 신뢰성 향상
   - 동일 정보를 여러 디스크에 중복 저장한다.
   - 하나의 디스크가 고장시 다른 디스크에서 읽어온다. (Mirroring, shadowing)
   - 단순한 중복 저장이 아니라 일부 디스크에 parity를 저장하여 공간의 효율성을 높일 수 있다.



<br/><br/>





















