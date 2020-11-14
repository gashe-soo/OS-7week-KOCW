# CH 8. Memory Management

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.

<br/><br/>

### CPU와 Memory

앞으로 메모리 관리에 다룰 것이다. 그 전에 간단히 두 가지를 복습해보고 가자.

<br/>

먼저, CPU가 프로세스를 실행할 때 어떻게 동작했는가를 떠올려보자. 

- 메모리에 올라와있는 프로세스에서 명령어를 레지스터로 가져온다.
- 명령어를 실행한다. 

여기서 CPU는 메인 메모리와 레지스터에 접근할 수 있다. 디스크에 바로 접근하지 않는다!

그렇기 때문에 **모든 실행되는 명령어와 데이터들은 CPU가 직접 접근할 수 있는 메인 메모리와 레지스터에 있어야 한다.**

<br/>

두 번째로, 프로세스는 어떻게 실행되는가.

알다시피, 프로세스는 실행하고 있는 프로그램을 뜻한다. 프로그램은 이진 실행 파일 형태로 디스크에 저장되어 있다. 

프로세스는 다음과 같이 여러 단계를 거친다. 

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img1.jpg?raw=true" alt="week5_ch8_img1.jpg" style="zoom:67%;" />

loader가 실행 파일을 메모리에 적재시켜 실행시킨다.

<br/>

간단히 말하면 프로세스는 일련의 과정을 통해서 메모리에 배치된다. 배치된 프로세스는 CPU가 직접 접근할 수 있다.

<br/>

### Logical vs Physical Address

메모리에 관련된 주소는 총 3가지이다. 

- Symbolic address
- Logical address (논리 주소)
- Physical address (물리 주소)

하나씩 얘기해보자. 

먼저 코드를 작성하면 변수를 할당하고 프로그램 내에서 변수에 접근한다. 즉, **특정 메모리 주소가 아닌 심볼 형태로 표현**되는 것이다. 이를 Symbolic address라고 한다.

이제 논리 주소와 물리 주소의 차이점을 알아보자.

논리 주소란 

- 프로세스마다 독립적으로 가지는 주소 공간을 말한다.
- 각 프로세스마다 0번지부터 시작한다.
- **CPU가 보는 주소는 logical address**이다. = 명령어를 확인하는데, 명령어에 들어가 있는 코드의 주소는 logical이기 때문이다.

<br/>

물리 주소란

- 메모리에 실제 올라가는 위치를 뜻한다.

<br/>

그럼 이걸 왜 나눴을까? 

뒤에서 자세히 다루겠지만, 간단히 말하면 메모리 관리를 위함이다. 

실행 중인 프로세스를 모두 메모리에 올리는 건 꽤나 메모리 낭비이다. 사용하지 않는 부분도 있기 때문이다. (프로세스 A라고 모든 코드를 실행하고 있는 건 아니다!) 그래서 최대한 공간 낭비를 하지 않기 위해 프로세스를 나누어 올린다. 나누어 올리는 것도 역시 공간 낭비하지 않기 위해 테트리스처럼 빈 곳에 넣어준다. 

그러면 물리 주소는 계속 바뀌게 될 것이다. 그런데 그걸 다 CPU나 프로세스가 기억하고 있을 수 있을까? 그러는 것은 낭비다. 그러므로 논리 주소라는 가상 주소 공간을 통해 CPU가 프로세스의 명령어를 가져오고 주소 변환을 통해 물리 주소에 접근하는 것이다. 

그럼 이제 주소 바인딩에 대해 알아보자.

<br/>

### 주소 바인딩

주소 바인딩이란 논리 주소를 물리 주소로 변환하는 것이다.

(Symbolic address -> Logical address -> Physical address)

그렇다면 논리적 주소에서 물리적 주소로 언제 결정되는가?

물리 메모리에 적재하는 방법은 총 3가지이다. 

<br/>

1. Compile time binding

- 물리적 메모리 주소(physical address)가 컴파일 시 정해진다 = 논리적 주소로 변환하면서 이미 물리적 주소도 정해짐
- 그러나 이미 주소가 정해지므로 시작 위치 변경시 재컴파일해야 한다.
- 컴파일러는 절대 코드(absolute code)를 생성한다.

<br/>

2. Load time binding

- loader의 책임하에 loading될 때 물리적 메모리 주소를 부여한다.
- 컴파일러가 재배치가능코드(relocatable code)를 생성한 경우 가능

<br/>

3. Execution time binding( = Runtime binding)

- 수행이 시작된 이후에도 프로세스의 **메모리 상 위치를 옮길 수 있다**.
- CPU가 주소를 참조할 때마다 binding을 점검한다. (address mapping table)
- 주소를 바꿀 수 있어야 하므로 하드웨어적인 지원이 필요하다. ( MMU, base and limit registers)

<br/>

어떻게 주소 바인딩하는 지 알아보자.

<br/>

### Memory Management Unit(MMU)

MMU(Memory Management Unit)

- logical address를 physical address로 매핑해주는 Hardware device

- 사용자 프로세스는 logical address만을 다룬다. 실제 physical address를 볼 수 없고 알 필요도 없다.
- MMU는 사용자 프로세스를 물리 주소로 변환해주는 역할을 한다.

<br/>

MMU가 어떻게 동작하는 지 간단한 예를 통해 살펴보자.

- MMU에 **운영체제 및 사용자 프로세스 간의 메모리 보호를 위해 사용**하는 2개의 레지스터가 있다.

  - Relocation register : 접근할 수 있는 물리적 메모리 주소의 최소값

  - Limit register : 논리적 주소의 범위

  - 예를 들어 프로세스 A의 논리 주소는 0~1000번지라고 하자. 물리 주소로 변환되니 14000번지부터 시작한다. 

    그렇다면 프로세스 A의 물리 주소 14000~15000번지일테고 relocation은 14000, limit register는 1000이 될 것이다.

  - 레지스터를 왜 사용할까?

    - 메모리 보호하기 위함이다. 만약 물리 주소의 범위 14000~15000인데, 16000에 접근하려고 한다면? 잘못된 주소를 접근하는 것이므로 이는 막아야 한다.

- 사용자 프로세스가 CPU에서 수행되며 생성해내는 모든 주소값에 대해 base register(= relocation register)의 값을 더한다.

<br/>

프로세스를 메모리에 올리는 데는 여러 방법이 있다. 

#### Dynamic Loading

- 프로세스 전체를 메모리에 미리 다 올리는 것이 아니라 해당 루틴이 불려질 때 메모리에 load한다.(Loading : 메모리로 올리는 것)
- 필요 루틴만 올라가므로 memory utilization 향상
- 가끔씩 사용되는 많은 양의 코드의 경우 유용 = 필요할 때만 올라가므로 메모리 낭비가 줄어든다!
  - EX. 오류 처리 루틴
- 운영체제의 특별한 지원 없이 프로그램 자체에서 구현 가능( **OS는 라이브러리를 통해 지원 가능**)


<br/>

#### Overlays

- 메모리에 프로세스의 부분 중 실제 필요한 정보만을 올린다.
- 프로세스의 크기가 메모리보다 클 때 유용하다.
- 운영체제의 지원 없이 사용자에 의해 구현한다.
- 작은 공간의 메모리를 사용하던 초창기 시스템에서 **수작업으로 프로그래머가 구현**
  - Manual overlay
  - 프로그래밍이 매우 복잡

> Dynamic Loading과의 차이점?
>
> = 운영체제의 지원 여부!

#### Swapping

이전에 process에서 suspended를 기억해보자. swapping을 통해 일부 프로세스를 메모리에서 쫓아냈다. 그것이 이거다.

**Swapping**

- 프로세스를 일시적으로 메모리에서 backing store로 쫓아 내는 것

**Backing store( = swap area)**

- 디스크 : 많은 사용자의 프로세스 이미지를 담을 만큼 충분히 빠르고 큰 저장 공간

**Swap in/out**

- 일반적으로 중기 스케줄러에 의해 swap out시킬 프로세스 선정
- priority-based CPU scheduling algorithm
  - priority가 낮은 프로세스를 swapped out 시킴
  - priority가 높은 프로세스를 메모리에 올려 놓음
- Compile time or load time binding에서는 원래 메모리 위치로 swap in 해야 한다.
- Execution time binding 에서는 추후 빈 메모리 영역 아무 곳에나 올릴 수 있음
- swap time은 대부분 transfer time(swap되는 양에 비례하는 시간)임

<br/>

#### Dynamic Linking

- Linking을 실행 시간(execution time)까지 미루는 기법
- Static linking
  - 라이브러리가 프로그램의 실행 파일 코드에 포함된다.
  - 실행 파일의 크기가 커진다.
  - 동일한 라이브러리를 각각의 프로세스가 메모리에 올리므로 메모리 낭비
- Dynamic linking
  - 라이브러리가 **실행시 연결**된다.
  - 라이브러리 호출 부분에 라이브러리 루틴의 위치를 찾기 위한 stub이라는 작은 코드를 둔다.
  - 라이브러리가 이미 메모리에 있으면 그 루틴의 주소로 가고 없으면 디스크에서 읽어온다.
  - 운영체제의 도움이 필요하다.

<br/>

### Allocation of Physical Memory

이제 메모리 할당을 어떻게 하는지 알아보자.

메모리는 일반적으로 두 영역으로 나뉘어 사용한다.

1. OS 상주 영역
   - interrupt vector와 함께 낮은 주소 영역 사용
2. 사용자 프로세스 영역
   - 높은 주소 영역 사용

<br/>

사용자 프로세스 영역의 할당 방법은 2가지이다.

- Contiguous allocation : 각각의 프로세스가 메모리의 연속적인 공간에 적재되도록 하는 것
  - Fixed partition allocation (고정 분할 방식)
  - Variable partition allocation (가변 분할 방식)
- Noncontiguous allocation : 하나의 프로세스가 메모리의 여러 영역에 분산되어 올라갈 수 있음 = 프로세스를 쪼개서 사용
  - Paging
  - Segmentation
  - paged segmentation

하나씩 자세히 살펴보자.

<br/>

### Contiguous allocation

연속 메모리 할당은 프로세스 모두가 연속적으로 올라가는 것을 의미한다.

<br/>

#### 고정 분할 방식

고정 분할 방식은 이름처럼 고정된 크기로 분할하는 방식이다. 

- 물리적 메모리를 몇 개의 영구적 분할(partition)으로 나눈다.
- 분할의 크기
  - 동일한 크기
  - 각자 다른 크기
- 분할당 하나의 프로그램를 적재한다.
- 융통성이 없다.
  - 동시에 메모리에 load되는 프로그램의 수가 고정될 수 밖에 없다. 
  - 최대 수행 가능 프로그램 크기 제한
- External/Internal fragmentation 발생

> Fragmentation?
>
> 메모리을 적재하면서 남는 공간을 의미한다.
>
> 예를 들어 총 메모리가 100이라고 하자. 20의 크기로 동일하게 분할한다면 총 5개의 partition이 생긴다.
>
> 5개의 프로세스를 올릴 수 있는데, 프로세스 1은 18의 크기를 가진다. 
>
> 그러면 partiton은 20이지만 프로세스 1은 18이므로 2만큼의 공간이 남는다. 할당된 프로세스가 분할된 메모리보다 작을 때 internal fragmentation이 생긴다.
>
> 반면 프로세스의 크기가 partition의 크기보다 크면 이는 적재될 수 없다. 4개의 프로세스가 적재되었고 나머지 프로세스가 모두 20보다 크다면 이는 적재될 수 없으므로 external fragmentation이 생긴 것이다.

<br>

#### 가변 분할 방식

- 프로그램의 크기를 고려해서 할당
- 분할의 크기, 개수가 동적으로 변한다.
- 기술적 관리 기법 필요하다. 
  - 왜 기술적 관리 기법이 필요한지는 아래에 설명할 compaction를 확인하자!
- External fragmentation 발생

<br>

**Hole**

- 가용 메모리 공간을 뜻한다.
- 다양한 크기의 hole들이 메모리 여러 곳에 흩어져 있다.
  - fragmentation이 발생하기 때문이다.
- 프로세스가 도착하면 수용가능한 hole을 할당
- 운영체제는 다음의 정보를 유지
  - 할당 공간
  - 가용 공간(hole)

<br>

**Dynamic Storage Allocation Problem**

가변 분할 방식에서 size n인 요청을 만족하는 가장 적절한 hole을 찾는 문제를 뜻한다. 

해결 방법에는 3가지가 있다.

- First-fit
  - Size가 n이상인 것 중 최초로 찾아지는 hole에 할당
- Best-fit
  - Size가 n 이상인 가장 작은 hole을 찾아서 할당
  - Hole들의 리스트가 크기순으로 정렬되지 않은 경우 모든 hole의 리스트를 탐색해야 한다.
  - 많은 수의 아주 작은 hole들이 생성된다.
- Worst-fit
  - 가장 큰 hole에 할당
  - 모든 리스트를 탐색
  - 상대적으로 아주 큰 hole들이 생성된다.
- First-fit과 Best-fit이 worst-fit보다 속도와 공간 이용률 측면에서 효과적이다.
  - 총 사이즈가 M일 경우
  - First-fit은 최선은 바로 찾는 경우 = O(1) / 최악 O(M)
  - Best-fit / Worst-fit 은 최선/최악 모두 O(M) (모든 리스트를 탐색해야 하므로)
  - 공간 이용률 측면에선 당연히 worst-fit이 큰 hole을 생성하므로 제일 낮다.

<br>

**Compaction**

- hole( = External fragmentation) 문제를 해결하는 방법이다.
- **사용 중인 메모리 영역을 한군데로 몰고 hole들을 다른 한 곳으로 몰아 큰 block으로 만드는 것**
- 매우 비용이 많이 든다.
- **최소한의 메모리 이동으로 compaction하는 방법(매우 복잡) => 어떤 프로세스를 이동할 것인지를 고려해야 한다.**
- compaction은 프로세스의 주소가 **실행 시간에 동적으로 재배치 가능한 경우에만 수행**될 수 있다.
- 위에서 언급한 가변 분할 방식의 기술적 관리 기법을 의미한다!

<br/>

<br>

### Noncontiguous Allocation

이번에는 연속적이지 않게 할당하는 방법이다. 

많은 이점을 제공하기에 현대 운영체제에서 사용한다.

<br>

#### Paging

프로세스를 동일한 사이즈의 page 단위로 나누어 관리하는 것이 paging의 핵심 개념이다.

나눈 page를 물리 메모리에 불연속하게 저장한다.

- 일부는 backing storage에, 일부는 physical memory에 저장한다.

<br>

**basic method**

- physical memory를 동일한 크기의 frame으로 나눈다.
- logical memory를 frame과 같은 크기의 page로 나눈다.
- 모든 가용 frame들을 관리한다.
- logical address를 physical address로 변환하기 위해 page table이란 것을 활용한다.
- external fragmentation 발생 하지 않지만, internal fragmentation 발생 가능하다.

<br>

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img2.jpg?raw=true" alt="week5_ch8_img2.jpg"  />

#### Paging Table

Page table은 프로세스의 논리 주소를 물리 주소를 변환시키기 위해 필요하다. 즉, 모든 프로세스는 각자의 table이 있어야 한다.

- Paging Table은 main memory에 상주한다.
- page table에는 page table entry가 여러 개 있다. 
  - page table entry : 논리 주소를 물리 주소로 맵핑해놓은 것.
- Page-table base register(PTBR)가 page table을 가리킴 = contiguous의 base register와 유사
- Page-table length register(PTLR)가 테이블 크기를 보관 = contiguous의 limit register와 유사
- 모든 메모리 접근 연산에는 2번의 memory access 필요
  - page table 접근 1번, 실제 data/instruction 접근 1번 => 시간 걸리므로 속도향상 필요
- 속도 향상을 위해 **associative register + translation look-aside buffer(TLB)라 불리는 고속의 lookup hardware cache 사용**

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img3.jpg?raw=true" alt="week5_ch8_img3.jpg"  />

<br>

associative register는 어떻게 빠를 수 있을까? 

 **Associative Register** 

- Parallel search 가능하다. 
  - 기존에는 순차적으로 검색했으므로 시간은 O(n)
  - 병렬 검색이 가능한 레지스터가 있다면 훨씬 빠르게 많은 양을 검색할 수 있다. = O(1)
- TLB에는 page table 일부만 존재한다.

**Address translation**

- page table 중 일부가 associative register에 보관되어 있다.
- 만약 해당 page가 associative register에 있는 경우 곧바로 frame을 얻는다.
- 그렇지 않은 경우 main memory에 있는 page table로부터 frame을 얻음
- TLB는 캐시이므로 프로세스의 주소를 계속 저장해 놓을 수 없다.
  - 그러므로 context switch때마다 flush한다.

<br>

**Access Time**

- Associative register lookup time = ε
- memory cycle time= 1
- Hit ratio = α
- Effective Access Time (EAT)
  
  - EAT = (1+ ε) α + (2 +  ε) (1 - α ) = 2 +  ε - α
  

<br/>

### Two-Level Page Table

현대의 컴퓨터는 address space가 매우 큰 프로그램을 지원한다. 32bit, 64bit 중에 32bit를 예를 들어보자.

- 32 bit address 사용시 2^32(=4G)의 주소 공간을 사용한다.
- page size가 4K시 1M개의 page table entry 필요
  - 각 page entry가 4B일 경우 프로세스당 4M의 page table 필요 => 공간이 엄청 필요하다.
  - 그러나, 대부분의 프로그램은 4G의 주소 공간 중 지극히 일부분만 사용하므로 page table 공간이 심하게 낭비된다.
    - 프로그램의 대부분은 현재 사용하지 않는다. 그런데 page table에는 올라와 있어야 한다. 당연히 p번째를 찾아야 하므로 중간이 비면 안된다!
  - 낭비를 막기 위해 two-level로 구성
    - page table 자체를 page로 구성
  - 사용되지 않는 주소 공간에 대한 outer page table의 엔트리 값은 NULL(대응하는 inner page table이 없음)
  

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img4.jpg?raw=true" alt="week5_ch8_img4.jpg"  />

#### Example

- logical address 구성 ( 32bit machine + 4K page size)

  - 20bit page number + 12 bit page offset
  - offset는 page size가 2^12이므로 12비트면 모두 구분 가능하다.

- page table 자체가 page로 구성되기 때문에 page number는 다음과 같이 나뉜다.

  - 10bit의 page number + 10 bit page offset

- logical address

  - | page number |      | page offset |
    | ----------- | ---- | ----------- |
    | p1          | p2   | d           |
    | 10          | 10   | 12          |

  - P1은 outer page table의 index이고 P2는 outer page table의 page에서의 d

<br>

### Multilevel Paging and Performance

- Address space가 더 커지면 다단계 page table이 필요하다.

- 각 단계의 page table이 메모리에 존재하므로 logical address의 physical address 변환에 더 많은 메모리 접근이 필요하다.

- TLB를 통해 메모리 접근 시간을 줄일 수 있다.

- 4단계 page table을 사용하는 경우

  - 메모리 접근 시간이 100ns, TLB 접근 시간이 20ns이고
  - TLB hit ratio가 98%인 경우
    - effective memory access time = 0.98*120 + 0.02 * 520 = 128ns
    - 결과적으로 주소 변환을 위해 28ns만 소요


<br/>

### Memory Protection

Page table의 각 entry마다 아래의 bit를 둔다

- Protection bit
  - page에 대한 접근 권한(read/write/read-only)을 의미한다.
    - code, data, stack에 해당하는 부분도 page에 올라갈 것이다.
    - EX. code에 해당하는 부분은 바뀌면 안된다. => read-only와 같이 권한을 두어 변경 방지
- Valid- invalid bit
  - valid 는 해당 주소의 frame에 그 프로세스를 구성하는 유효한 내용이 있음을 뜻함(접근 허용)
  - invalid는 해당 주소의 frame에 유효한 내용이 없음을 뜻함(접근 불허)
    - 프로세스가 그 주소 부분을 사용하지 않는 경우
    - 해당 페이지가 메모리에 올라와 있지 않고 swap area에 있는 경우

<br/>

### Inverted Page Table

page table의 가장 큰 단점은 page table의 공간이 너무 크다는 것이다. 

inverted page table은 그러한 점을 해결한다.

- Page table이 매우 큰 이유
  - 모든 process별로 그 logical address에 대응하는 모든 page에 대해 page table entry가 존재
  - 대응하는 page가 메모리에 있든 아니든 간에 page table에는 entry로 존재

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img5.jpg?raw=true" alt="week5_ch8_img5.jpg"  />

Inverted page table
- 원래는 프로세스의 page table을 보고 physical memory의 주소를 찾아간다.
- 그러나 **inverted는 반대로 frame의 개수만큼 page table을 생성! 그래서 f만큼 떨어진 곳에 주소를 둔다.**
- page frame 하나당 page table에 하나의 entry를 둔 것(system-wide)
- 각 page table entry는 각각의 물리적 메모리의 page frame이 담고 있는 내용 표시 (process-id, process의 logical address)
- **메모리에 올라와있는 프로세스만 가지고 있기에 공간 낭비 막을 수 있다**.
- 논리적 주소 -> 물리적 주소로 변환해야 하지만, page table은 f만큼 떨어진 곳에 있고 우리는 f를 모르고 p만 안다. 따라서 pid +p로 page table를 모두 검색해야 찾을 수 있다.
- 단점
  - 테이블 전체를 탐색해야 함
- 조치
  - associative register 사용(expensive) => TLB처럼 동시에 여러 개 탐색

<br/>

### Shared Page

하나의 프로그램을 여러 개 실행하면 어떻게 메모리에 올려야 할까?

같은 프로그램이므로 코드는 다 같은데, 그 안에서 실행되는 데이터만 다를 것이다. 

그런데 코드에 해당하는 부분을 모두 각자 메모리에 배치해야 하는 것은 낭비일 것이다. 어차피 하는 일은 똑같은데 말이다.

그래서 page를 공유하는 것이다. 어차피 물리 주소만 똑같이 접근하게 하면 된다. 

- Shared code
  - Re-entrant code( = Pure code)
  - **read-only로 하여 프로세스 간에 하나의 code만 메모리에 올림**
    - ex. text editors, compilers
  - shared code는 모든 프로세스의 logical address space에서 동일한 위치에 있어야 한다.
- Private code and data
  - 각 프로세스들은 독자적으로 메모리에 올림
  - Privated data는 logical address space의 아무 곳에 와도 무방

<br>

### Segmentation

paging에서는 frame 단위로 나눠서 넣도록 했다.

- 프로그램은 의미 단위인 여러 개의 segment로 구성
  - 작게는 **프로그램을 구성하는 함수 하나하나를 세그먼트로 정의**
  - 크게는 프로그램 전체를 하나의 세그먼트로 정의 가능
  - 일반적으로는 code, data, stack 부분이 하나씩의 세그먼트로 정의됨
- segment는 다음과 같은 **logical unit**들임
  - main()
  - function
  - global variables
  - stack
  - symbol table, arrays

<br/>

### Segmentation Architecture

- Logical address는 다음의 두 가지로 구성
  - < segment-number, offset> 
- Segment table
  - each table entry has :
    - base : 세그먼트가 시작하는 물리 주소
    - limit : 세그먼트의 길이
- Segment-table base register(STBR)
  - 물리적 메모리에서의 segment table의 위치
- Segment-table length register (STLR)
  - 프로그램이 사용하는 segment의 수

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img6.jpg?raw=true" alt="week5_ch8_img6.jpg"  />

1. 먼저 s을 이용해 segment table에서 limit을 가져와 d와 비교한다. 만약 d가 limit보다 크다면 이는 잘못된 주소 참조이므로 trap을 발생시킨다.
2. 1번에서 trap이 발생하지 않았다면 base + d를 하여 물리 주소를 확인한다.

<br>

- Protection
  - 각 세그먼트 별로 protection bit이 있음
  - Each entry : 
    - valid bit = 0 => illegal segment
    - Read/Write/Execution 권한 bit
- Sharing
  - shared segment
  - same segment number
  - segment는 **의미 단위이기 때문에 공유(sharing)와 보안(protection)에 있어 paging 보다 훨씬 효과적**이다.
- Allocation
  - first fit / best fit
  - external fragmentation 발생
  - segment의 길이가 동일하지 않으므로 가변분할 방식에서와 동일한 문제점들이 발생

<br>

segment table에는 segment수만큼 entry가 있다.

<br>

### Segmentation with Paging

segmentation뿐만 아니라 paging도 함께 사용한다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img7.jpg?raw=true" alt="week5_ch8_img7.jpg"  />

[출처 : 강의]

<br/>

- 기본적으로 segment 단위로 나눈다.
- 나눠진 segment를 다시 page 단위로 나눠 관리한다.
- pure segmentation과의 차이점
  - segment-table entry가 segment의 base address를 가지고 있는 것이 아니라 segment를 구성하는 page table의 base address를 가지고 있다.

- 장점 : allocation의 문제가 없다. 

<br/><br/>