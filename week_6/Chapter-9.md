# CH 9. Virtual Memory

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.

<br/><br/>

### Background

이전 단원에서 페이징과 세그멘테이션에 대해 공부했다. 

주로 생기는 문제점은 무엇이었냐하면 공간 낭비였다. 주로 사용하지 않는 코드를 메모리에 올려놓으면 메모리만 잡아먹고 쓸모가 없는 것이다.

그래서 이런 생각을 하게 되었다.

- 프로그램을 필요한 부분만 메모리에 올려놓고 실행한다면?

이것이 바로 가상 메모리(Virtual Memory)이다.

가상 메모리는 실제의 물리 메모리 개념과 개발자의 논리 메모리 개념을 분리한 것이다. 

한 프로세스의 가상 주소 공간은 프로세스가 메모리에 저장되는 논리적인 모습을 말한다. 

<br>

### Demand Paging

위에서 말했듯이 **필요한 페이지만 메모리에 올리고 사용하는 기법이 요구 페이징(demand paging)**이다.

#### Valid/Invalid bit의 사용

이전에 페이징에서 valid/invalid bit를 떠올려보자.

valid하다면 이는 메모리에 적재되어 있는 것이고,

invalid하다면 사용되지 않는 주소 영역이거나, 물리적 메모리에 없는 경우이다. 

그렇다면 물리적 메모리에 없다면 어디에 있을까? backing store(또는 disk)라고 할 수 있다. 

처음에 page table에서 page entry를 invalid로 초기화한다. 

주소 변환하기 위해 page entry에 접근했을 때 invalid bit라면 이는 backing store에서 페이지를 가져와야 한다. 

이를 **page fault trap**이라 한다.

왜 trap이냐면, backing store에 페이지가 있는데 이를 메모리에 가져오는 것은 I/O다. ( = 디스크 I/O) 그렇기 때문에 OS에 CPU를 넘겨서 I/O를 처리하도록 해야 하는 것이다.

<br>

그렇다면 demand paging의 효과는 무엇일까?

- I/O 양의 감소 => 필요한 부분만  올리므로

- Memory 사용량 감소
- 빠른 응답 시간
- 더 많은 사용자 수용

가 있다.

<br>

### Page Fault

위에서 다루었던 page fault에 대해 좀 더 자세히 알아보자.



<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img1.jpg?raw=true" alt="week6_ch9_img1.jpg"/>

- Invalid page를 접근하면 MMU가 trap을 발생시킨다. ( =  page fault trap)
- Kernel mode로 들어가서 page fault handler가 invoke된다.
- 다음과 같은 순서로 page fault를 처리한다.
  1. Invalid reference? (ex. bad address, protection violation)=> abort process
  2. Get an empty page frame 
     - 만약 빈 페이지가 없으면 있는 걸 disk로 쫓아내고 그 자리 차지 = replace
  3. 해당 page를 disk에서 memory로 읽어온다.
     1. disk I/O가 끝나기까지 이 프로세스는 CPU를 preempt 당함( block)
        - I/O는 시간이 오래 걸린다. 그러므로 disk controller에게 I/O 시키고 다른 일 하기.
     2. Disk read가 끝나면 page table entry 기록, valid/invalid bit = "valid"
        - I/O 끝나면 interrupt 걸어서 valid 처리해주고
     3. ready queue에 process를 insert => dispatch later
  4. 이 프로세스가 cpu를 잡고 다시 running
  5. 아까 중단되었던 intsruction을 재개



<br>

#### Performance of Demand Paging

- Page fault Rate 0<=p<=1

  - if p = 0, no page faults
  - if p = 1, every reference is a fault

- Effective Access Time (EAT)

  =  (1-p) * memory access  + p ( OS & HW page fault overhead + [swap page out if needed] + swap page in + OS&HW restart overhead)

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img2.jpg?raw=true" alt="week6_ch9_img2.jpg"/>



EAT 계산식이 복잡해보인다. 자세히 보면,

먼저 (1-p) * memory access 이 부분은 page fault가 나지 않으면 메모리에 적재되어 있는 것이므로 memory access하면 끝이다.

그 뒤에는 page fault가 발생했을 경우이다.

괄호 안의 식을 보자.

- OS & HW page fault overhead = trap이 발생하면 cpu를 os에 넘기고 disk controller에게 일을 시키는 과정이다. 

- [swap page out if needed] + swap page in => 만약 메모리가 이미 가득 차있다면 swap in을 해도 넣을 공간이 없으므로 빼야 한다. 그래서 빼는 비용 + 넣는 비용을 뜻한다.
- OS&HW restart overhead => 위와 동일하게 다시 CPU가 process에 넘어오는 과정이다.

<br>



### Page replacement

Page replacement는 단어 그대로 page를 교체하는 것을 뜻한다. 즉, 어떤 frame을 빼앗아올지 결정하는 것이다. 

그렇다면 어떤 것을 골라서 swap out시켜야 할까?

- 제일 오랫동안 사용되지 않은 페이지?
- 곧바로 사용되지 않을 페이지?

이런 것을 고르는 것이 Replacement Algorithm이다.

Replacement Algorithm의 목표는 page fault rate를 최소화하는 것이다. (EAT 계산을 생각해보자)

알고리즘을 평가하는 방법은 주어진 page reference string에 대해 page fault를 얼마나 내는지 조사하는 것이다.

reference string의 예

- 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

<br>

### Optimal Algorithm

제일 이상적인 방법은 가장 먼 미래에 참조되는 page를 교체하는 것이다.

예를 들어보자. 4개의 frame이 있다고 가정한다.

reference string이 다음과 같다.

- 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img3.jpg?raw=true" alt="week6_ch9_img3.jpg"/>

page fault인 경우 빨강, 그렇지 않을 경우 파랑으로 표시했다.

순서대로 확인해보자.

- 처음 1,2,3,4는 fault가 난다. 

- 이후, 1,2는 이미 존재하므로 파랑
- 5번은 1,2,3,4 중 가장 먼 미래에 참조되는 4번을 대체해서 들어간다.
- 1,2,3은 존재하므로 파랑으로 표시
- 4번은 page fault가 났다. 이제 1,2,3 중에 하나를 대체해서 들어간다.
- 마지막으로 5번은 존재.

<br>

그런데 미래의 참조를 실제로 알 수 있을까? 

답변은 그렇지 못하다. 

그래서 optimal algorithm은 다른 알고리즘의 성능에 대한 upper bound가 된다.

- Belady's optimal algorithm, MIN, OPT 등으로 불림

<br>

### FIFO Algorithm

FIFO(First In First Out) : 먼저 들어온 것을 먼저 내쫓는다. 

예를 들어보자. 3개의 frame이 있다고 가정한다.

reference string이 다음과 같다.

- 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img4.jpg?raw=true" alt="week6_ch9_img4.jpg"/>

FIFO의 독특한 점은 frame수가 늘어나면 당연히 page fault가 줄어들어야 한다고 생각하지만, fifo을 사용하면 그렇지 않은 경우가 있다.

이를 FIFO Anomaly(Belady's Anomaly)라 한다.

<br>

### LRU 알고리즘

Least Recently Used : 가장 오래 전에 참조된 것을 지운다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img5.jpg?raw=true" alt="week6_ch9_img5.jpg"/>

LRU는 가장 오래 전에 참조된 것을 지운다. 그러면 어떻게 구현하면 될까?

구현하면 중요한 것은 시간의 순서다. 다만 시간의 순서를 확보하기 위해서는 하드웨어의 지원이 필요하다.

doubly linked list 형태로 page의 시간 순서대로 연결하면 가능하다. 

제일 오래 전에 참조된 것은 head일테고, 가장 최근에 참조된 것은 tail일 것이다. 

따라서 O(1)의 시간복잡도를 가진다. 

> 어떻게 O(1)일까?
>
> doubly linked list에서 특정 값을 검색하는데 O(n)이 걸린다
>
> 중요한 것은 LRU 알고리즘은 하드웨어의 지원이 필요하다는 사실이다.
>
> 각 node의 주소를 저장하는 TLB레지스터와 같은 HW이 있다면 O(1)의 시간으로 node에 접근 가능하다. 
>
> 그러므로 삭제도 O(1)에 가능한 것이다!



<br>

### Clock Algorithm

LRU는 언급했다시피 하드웨어의 지원이 필요하다. 

그런데 하드웨어의 지원이 충분치 않으면 어떻게 해야할까?

이를 보완한 알고리즘이 바로 Clock Algorithm이다.

Clock Algorithm은 LRU의 근사 알고리즘이며 Second Chance Algorithm이라고도 불린다. (NUR(Not Used Recently) 또는 NRU(Not Recently Used))

<br>

#### 동작 원리

해당 알고리즘의 기본은 FIFO 알고리즘이다. 다만 참조 비트(reference bit)을 사용해서 교체 대상 페이지를 선정한다.

다음과 같이 동작한다.

0. 클락 알고리즘은 순환하면서 교체할 페이지를 찾는다.

1. 페이지가 선택될 때마다 참조 비트를 확인한다. 
   - 0이면 교체, 1이면 다시 기회를 주고 다음 포인터로 이동 

2. 포인터 이동하는 중에 reference bit 1은 모두 0으로 바꾼다.

3. 한 바퀴 되돌아와서도 0이면 그때는 replace한다. ( 한 번 더 기회를 주기 때문에 second chance이다! )
   - 자주 사용되는 페이지라면 second chance 올 때 1일 것이다.

이를 순환 큐를 이용하여 구현한다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img7.jpg?raw=true" alt="week6_ch9_img7.jpg" style="zoom:67%;" />

<br>

#### Clock Algorithm의 개선

reference bit과 modified bit(dirty bit)을 함께 사용하여 개선할 수 있다.

- reference bit =1 : 최근에 참조된 페이지
- modified bit = 1 : 최근에 변경된 페이지 ( I/O를 동반하는 페이지)

이렇게 2개의 비트가 있으므로 총 4가지 경우가 발생한다.

<br>

> 🔎**단순한 클락 알고리즘과의 차이점은 무엇일까?**
>
> 최근에 변경된 페이지를 교체하면 I/O를 동반한다. 
>
> 단순한 클락 알고리즘은 변경되든 말든 최근에 참조되지 않은 페이지를 교체하려고 한다.
>
> 그런데 개선된 알고리즘은 변경된 경우도 파악하여 **I/O를 줄일 수 있다.**
>
> 즉, 둘의 가장 큰 차이는 I/O 횟수다.

<br>

### LFU 알고리즘

Least Frequently Used : 참조 횟수가 가장 적은 페이지를 지운다.

- 최저 참조 횟수인 page가 여러 개인 경우
  - LFU 알고리즘 자체에서는 여러 page 중 임의로 선정한다.
  - 성능 향상을 위해 가장 오래 전에 참조된 page를 지우게 구현할 수 있다.
- 장단점
  - LRU처럼 직전 참조 시점만 보는 것이 아니라 장기적인 시간 규모를 보기 때문에 page의 인기도를 좀 더 정확히 반영할 수 있다.
  - 참조 시점의 최근성을 반영하지 못한다.
  - LRU보다 구현이 복잡하다.
    - min heap으로 구현
    - 쫓아내는 건 root
    - O(logn) 

<br>

#### EXAMPLE

1,1,1,1,2,2,3,3,2,4,5

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img6.jpg?raw=true" alt="week6_ch9_img6.jpg"/>

현재 시각 5번 page를 보관하기 위해 어떤 page를 삭제해야 하는가?

- LRU : 1번
- LFU : 4번

<br>

> **Paging system에서 LRU & LFU를 쓸 수 있을까?**
>
> CPU가 프로세스를 접근하면서 page의 물리 주소를 요청했다고 가정하자.
>
> 만약 요청한 page가 valid하다면 CPU는 바로 물리 주소에 접근할 수 있다.
>
> invalid하다면 trap이 걸려서 OS에 제어권이 넘어가 swap in을 해준다. 
>
> 여기서 LRU, LFU는 모두 참조횟수나 최근 참조된 page를 알고 있어야 한다.
>
> 그런데 OS는 그 횟수를 알고 있을까? 
>
> 정답은 그렇지 않다. 왜냐면 OS에 제어권이 넘어오는 경우는 page fault가 일어나는 경우만 해당하기 때문이다.
>
> 만약 fault가 계속 일어나지 않았다면? 참조 횟수는 계속 올라가는데 OS는 확인할 수 없다는 것이다. 이는 HW적인 움직임이기 때문이다.

<br/>



### 다양한 캐시 환경

- 캐시 기법
  - 한정된 빠른 공간에 요청된 데이터를 저장해 두었다가 후속 요청시 캐시로부터 직접 서비스하는 방식
  - paging system 외에도 cache memory, buffer caching, web caching 등 다양한 분야에서 사용
- 캐시 운영의 시간 제약
  - 교체 알고리즘에서 삭제할 항목을 결정하는 일에 지나치게 많은 시간이 걸리는 경우 실제 시스템에서 사용할 수 없다.
  - Buffer caching이나 web caching의 경우
    - O(1) ~ O(logn) 허용
  - Paging system 인 경우
    - page fault인 경우에만 OS가 관여함
    - 페이지가 이미 메모리에 존재하는 경우 참조시각 등의 정보를 OS가 알 수 없다.
    - O(1)인 LRU의 list 조작조차 불가능



<br>

### Page frame의 allocation

위에서 Paging에 대해서 어느정도 다루었다.

그렇다면 각 process에 얼마만큼의 page frame을 할당해야 할까?

그 전에, 왜 process마다 frame을 할당해줘야 할까?

- Allocation의 필요성 
  - 메모리 참조 명령어 수행시 명령어, 데이터 등 여러 페이지를 동시에 참조할 수 있다.
    - 명령어 수행을 위해 최소한 할당되어야 하는 frame의 수가 있다.
  - Loop를 구성하는 page들은 한꺼번에 allocate 되는 것이 유리함
    - 최소한의 allocation이 없으면 매 loop마다 page fault

- Allocation Scheme
  - Equal Allocation : 모든 프로세스에 똑같은 개수 할당
  - Proportional allocation : 프로세스 크기에 비례하여 할당
  - Priority allocation : 프로세스의 priority에 따라 다르게 할당

<br>

### Global vs Local Replacement

Frame을 할당하는 방법에는 또 다른 중요한 요소가 있다. 

'어느 곳에서 frame을 뺏어와 대체할 것인가'이다.

두 가지 범주가 있다.

1. Global replacement = 전역 교체
   - **Replace 시 다른 process에 할당된 frame을 빼앗아 올 수 있다.**
   - process별 할당량을 조절하는 또 다른 방법이다.
   - FIFO, LRU, LFU 등의 알고리즘을 global replacement로 사용시에 해당
   - Working set, PFF 알고리즘 사용

<br>

2. Local replacement = 지역 교체
   - **자신에게 할당된 frame 내에서만 replacement**
   - FIFO, LRU, LFU 등의 알고리즘을 process 별로 운영시

<br>

### Thrashing

만약에 전역 교체 알고리즘을 사용하는데, 프로세스가 많아지면 어떻게 될까?

프로세스가 많아지면 할당되는 page frame 수가 낮아지므로 page fault rate올라가고 swap in 해야하는 I/O가 발생할 것이다.

=> I/O가 많아진다는 것은? =  CPU 이용률 저하된다.

<br>

0. OS는 CPU의 utilization(이용률)을 확인하면서 낮아지면 다중 프로그래밍의 정도(MultiProgramming Degree)를 높인다. 

1. 프로세스가 시스템에 추가된다. = higher MPD
2. 프로세스의 원활한 수행에 필요한 최소한의 page frame 수를 할당 받지 못한 경우 발생 

3. page fault rate 상승 

4. CPU 이용률 저하

<br>

이렇게 하나의 cycle이 발생하는 것이다.

**이처럼 일정 MPD를 넘어버리는 순간 utilization이 낮아지는 것을 Thrashing이라 한다.**

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img8.jpg?raw=true" alt="week6_ch9_img8.jpg"/>

이제 Thrashing을 해결하는 방법을 보자.

<br>

### Working Set Model

첫 번째로는 working set model이다. 

프로세스는 특정 시간 동안 일정 지역만을 집중적으로 참조한다. 이를 Locality of reference라고 부른다. 지역성 모델(locality model)이라고도 한다.

working set model를 지역성을 활용한다.

Locality에 기반하여 프로세스가 일정 시간 동안 원활하게 수행되기 위해 한꺼번에 메모리에 올라와 있어야 하는 page들의 집합을 **Working Set**이라 정의한다.

Working set 모델에서는 process의 working set 전체가 메모리에 올라와 있어야 수행되고 그렇지 않을 경우 모든 frame을 반납한 후 swap out (suspend)한다.

= 전체가 보장 안되면 나중에 실행하겠다.

Multiprogramming degree를 결정하는 역할을 한다.

<br>

#### Working set Algorithm

- Working set의 결정

  - Window set window를 통해 결정한다.

  - window size가 Δ인 경우

    - 시각 t에서의 working set WS(t) = Time Interval [ t - Δ, t] 사이에 참조된 서로 다른 페이지들의 집합

    - Working set에 속한 page는 메모리에 유지, 속하지 않은 것은 버림

      (즉, 참조된 후 Δ 시간동안 해당 page를 메모리에 유지한 후 버림)

  - 정확도는 window size에 따라 정해진다.
  
    - size가 너무 크면 여러 지역성을 과도하게 수용한다. 
    - size가 너무 작으면 전체 지역을 포함하지 못한다.

<br>

### PFF(Page Fault Frequency) Scheme

Thrashing은 page fault rate이 높아서 발생한다. 

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week6_ch9_img9.jpg?raw=true" alt="week6_ch9_img9.jpg"/>

그래서 Page fault rate의 상한값과 하한값을 둔다
- Page fault rate이 상한값을 넘으면 frame을 더 할당한다.
- Page fault rate이 하한값 이하이면 할당 frame 수를 줄인다

- 빈 frame이 없으면 일부 프로세스를 swap out

<br>

### Page size의 결정

- page size를 감소시키면
  - 페이지 수 증가
  - 페이지 테이블 크기 증가
  - Internal fragmentation 감소
  - Disk transfer의 효율성 감소
    - seek/rotation vs transfer
  - 필요한 정보만 메모리에 올라와 메모리 이용이 효율적
    - Locality의 활용 측면에서는 좋지 않음
      - page fault가 늘어나므로!
- Trend
  - Larger page size