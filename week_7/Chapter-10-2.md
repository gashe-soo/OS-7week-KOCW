# CH 10-2. File Systems Implementation

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.

<br/>

<br/>

### Allocation of File Data in Disk

disk에 파일을 저장할 때 어떻게 저장할 것인가에 대한 방법이 3가지 있다.

#### Contiguous Allocation

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch10_img1.jpg?raw=true" alt="week6_ch9_img1.jpg"/>

- 그림과 같이 연속적인 메모리에 올리는 방법이다.
- 장점
  - Fast I/O
    - 한 번의 seek/rotation으로 많은 byte transfer 가능하다.
    - Realtime file용으로 또는 이미 실행 중이던 process의 swapping 용
  - Direct access(= random access) 가능하다.
- 단점
  - 외부 단편화가 발생한다.
  - File grow가 어렵다. (grow란 수정 등으로 인해 파일의 크기가 변하는 것을 뜻한다.)
    - file 생성시 얼만큼의 hole를 배정할 것인가?
    - Grow vs 낭비(Internal fragmentation)



<br/>

#### Linked Allocation

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch10_img2.jpg?raw=true" alt="week7_ch10_img2.jpg"/>

- 각 블록이 다음 블록의 주소를 가지고 있다. 디렉토리는 파일의 시작과 끝 위치만 가지고 있다.

- 장점
  - 외부 단편화가 발생하지 않는다.
- 단점
  - Direct access가 불가능하다. 
  - Reliability 문제
    - 한 sector가 고장나 pointer가 유실되면 많은 부분을 잃는다.
  - Pointer를 위한 공간이 block의 일부가 되어 공간 효율성을 떨어뜨림
    - 512 bytes/sector, 4 bytes/pointer 라고 한다면 사실상 데이터를 저장할 수 있는 공간은 512-4 = 508 bytes다.
    - 만약 512 bytes file을 저장하려면? 2sector를 사용해야 한다. => 공간 낭비
- 변형
  - File Allocation Table(FAT) 파일 시스템
    - pointer를 별도의 위치에 보관하여 reliability와 공간효율성 문제 해결

<br/>

#### Indexed Allocation

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch10_img3.jpg?raw=true" alt="week7_ch10_img3.jpg"/>

- Index block을 두어 파일의 내용을 가지고 있는 주소를 저장해둔다.
- 그림의 경우 총 5개의 블록으로 이루어져 있다.

-  장점
  - 외부 단편화가 발생하지 않는다.
  - Direct access 가능
- 단점
  - Small file의 경우 공간 낭비 (실제로 많은 file이 small)
    - 만약 블록이 1개인 파일을 저장하려면 index block를 저장해야 하므로 총 2개의 sector나 필요하다.
  - 반대로 Too large file의 경우 하나의 block으로는 index를 저장하기에 부족하다.
    - 해결 방안
      1. linked scheme = index block의 마지막을 새로운 index block의 주소로 표시
      2. multi-level index = 계층화

<br/>

### Unix File System Architecture

UNIX 의 파일 시스템은 크게 4가지로 구성되어 있다.

1. Boot block
   - 부팅에 필요한 정보 (bootstrap loader)
2. Super block
   - 파일 시스템에 관한 총체적인 정보를 담고 있다.
   - 어디가 사용 중인지, 어디까지가 inode list인지 등의 정보를 관리
3. Inode list
   - **파일 이름을 제외**한 파일의 모든 meta data를 저장
   - <img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch10_img4.jpg?raw=true" alt="week7_ch10_img4.jpg"/>
   - direct blocks + indirect 을 이용하여 모든 파일의 meta data를 저장할 수 있다.
4. Data block
   - 파일의 실제 내용을 보관
   - file이름 + inode 번호를 보관

<br/>

### FAT File system

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch10_img5.jpg?raw=true" alt="week7_ch10_img5.jpg"/>

1. Boot block 

2. FAT

   - FAT는 Linked Allocation을 활용한 것이다. 다만, pointer를 별도의 위치에 보관하여 reliability와 공간효율성 문제를 해결했다.
   - 테이블을 따로 만들어 저장하며 테이블의 크기는 데이터 블록의 개수와 동일하다. 만약 N개의 데이터 블록이 있다면 table의 크기는 N이다.
   - reliability가 중요하므로 copy가 있다.

3. Root directory

4. Data block

   - UNIX와는 다르게 file이름 등 파일 정보를 data block에 저장한다. 또한, data block의 첫 번째 주소를 저장한다.

     <br/>

### Free-Space Management

이제부터는 파일을 할당받지 못하고 남은 데이터 블록을 어떻게 관리하는 지에 대해 알아보자.

#### Bit map or bit vector

- bit[i] 은 block free일 경우 0, occupied일 경우 1이다.

  | 0    | 1    | 2    | ...  | ...  | n-1  |
  | ---- | ---- | ---- | ---- | ---- | ---- |
  | 1    | 0    | 1    |      |      | 1    |

- bit map은 부가적인 공간을 필요로 한다.

- 연속적인 n개의 free block을 찾는데 효과적이다.

  <br/>

#### Linked List

- 모든 free block들을 링크로 연결(free list)
- 연속적인 가용공간을 찾는 것은 쉽지 않다.
- 공간의 낭비가 없다.

<br/>

#### Grouping

- linked list 방법의 변형 + index를 이용
- 첫 번째 free block이 n개의 pointer를 가진다.
  - n-1 pointer는 free data block을 가리킨다.
  - 마지막 pointer가 가리키는 block은 또 다시 n pointer를 가진다.

<br/>

#### Counting

- 프로그램들이 종종 여러 개의 연속적인 block을 할당하고 반납한다는 성질에 착안
- (first free block, # of contiguous free blocks)를 유지

<br/>

### Directory Implementation

#### Linear list

- <file name, file metadata> 의 형태로 배열로 저장한다.
- 구현이 간단하다.
- 디렉토리 내에 파일이 있는지 찾기 위해서는 linear search가 필요하다. ( time consuming)

<br/>

#### Hash Table

- Linear list + hashing
- Hash table은 file name을 이 파일의 linear list의 위치로 바꾸어 준다.
- search time을 없앴다.
- Collision 발생 가능

<br/>

#### File metadata의 보관 위치

- 디렉토리 내에 직접 보관
- 디렉토리에는 포인터를 두고 다른 곳에 보관
  - Inode, FAT 등

<br/>

#### Long file name의 지원

- <File name, file metadata>의 list 에서 각 entry는 일반적으로 고정 크기
- file name이 고정 크기보다 길어지는 경우 entry의 마지막 부분에 이름의 뒷부분이 위치한 곳의 포인터를 두는 방법
- 이름 나머지 부분은 동일한 directory file의 일부에 존재

<br/>

### VFS & NFS

#### VFS(Virtual File System)

- 서로 **다른 다양한 file system에 대해 동일한 시스템 콜 인터페이스(API)를 통해 접근**할 수 있게 해주는 OS layer

<br/>

#### NFS(Network File System)

- 분산 시스템에서는 **네트워크를 통해 파일이 공유**될 수 있다.
- NFS는 분산 환경에서의 대표적인 파일 공유 방법이다.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch10_img6.jpg?raw=true" alt="week7_ch10_img6.jpg"/>



<br/>

### Page cache and Buffer Cache

- **Page Cache**

  - Virtual memory의 paging system에서 사용하는 page frame을 caching의 관점에서 설명하는 용어
  - Memory-mapped I/O를 쓰는 경우 file의 I/O에서도 page cache 사용

  <br/>

- **Memory Mapped I/O**

  - File의 일부를 virtual memory에 mapping 시킨다.
  - 매핑시킨 영역에 대한 메모리 접근 연산은 파일의 입출력을 수행하게 한다.

  <br/>

- **Buffer Cache**

  - 파일 시스템을 통한 I/O 연산은 메모리의 특정 영역인 buffer cache 사용
    - 먼저 buffer cache로 읽어온 후 사용자 프로세스에 전달
  - File 사용의 locality 활용
    - 한 번 읽어온 block에 대한 후속 요청시 buffer cache에서 즉시 전달
  - 모든 프로세스가 공용으로 사용
  - Replacement algorithm 필요

  <br/>

- **Unified Buffer Cache**

  - 최근의 OS에서는 기존의 buffer cache가 page cache에 통합되었다.
  - <img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week7_ch10_img7.jpg?raw=true" alt="week7_ch10_img7.jpg"/>

<br/><br/><br/>







### REFERENCE

[dbms-notes](http://www.dbms-notes.com/2010/10/network-file-system-nfs.html)