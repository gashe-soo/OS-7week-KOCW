# CH 6-2. Problem with Semaphore

다음은 KOCW에서 제공한 반효경 교수님의 [운영체제](http://www.kocw.net/home/search/kemView.do?kemId=1046323)를 정리한 글입니다.

+ 공룡책이라 불리는 '운영체제(Operating System Concepts) 10th edition'을 보고 내용을 추가해 정리한 글입니다.

<br/><br/>

### Bounded-Buffer Problem(Producer-Consumer Problem)

생산자- 소비자 문제는 n개의 buffer를 가진 pool를 공유 메모리로 접근한다.

- 생산자는 데이터를 생산해서 buffer에 넣는다.
- 소비자는 pool의 buffer에서 데이터를 꺼내온다.

<br/>

여기서 주의해야 할 점이 있다.

1. Synchronized와 관련 가능한 문제?

   - 생산자 두 명이 동시에 데이터를 같은 버퍼에 넣으려고 하면 문제 => lock를 걸어서 진행
   - 소비자 두 명이 동시에 동일한 버퍼의 데이터에 접근하면 문제

2. 버퍼가 유한하기 때문에 생기는 문제

   - pool의 버퍼가 다 채워졌을 경우 생산자는 하나가 비워질 때까지 기다려야 한다.
   - pool의 버퍼가 다 비워져있을 경우 소비자는 채워질 때까지 기다려야 한다.


<br/>

이제 코드를 보자. 

Shared data

- Buffer 자체 및 buffer 조작 변수(empty/full buffer의 시작 위치)

Synchronization variables

- Mutual exclustion -> need binary semaphore
- Resource count -> need integer semaphore

```c
// Synchronization variables
semaphore full = 0, empty = n, mutex = 1;

//Producer
do{
	produce an item in X
	...
	P(empty);	// empty 버퍼가 있는 지 확인한다. 없으면 기다림
	P(mutex);
	...
	add x to buffer
	...
	V(mutex);
	V(full);	// 버퍼에 넣었으니 full 증가
}while(1);

// Consumer
do{
	P(full)
	P(mutex);
	...
	remove an item from buffer to y
	...
	V(mutex);
	V(empty);
	...
	consume the item in y
	...
}while(1);
```

다음은 monitor를 이용한 소스코드이다.

```c
monitor bounded_buffer{
    int buffer[N];
    condition full,empty;
    
    void produce(int x){
        if there is no empty buffer
            empty.wait();
        add x to an empty buffer;
        full.signal();
    }
    
    void consume(int *x){
        if there is no full buffer
            full.wait();
        remove an item from buffer and store it to *x;
        empty.signal();
    }
}
```



### Reader- Writers Problem

- **한 process가 DB에 write 중일 때 다른 process가 접근하면 안 된다.** 
  - 만약 쓰고 있는데 그것을 읽는 process가 온다면 잘못된 데이터를 읽어갈 수도 있다. 
  - 혹은 writer-writer일 경우 당연히 데이터 무결성 보장하지 못한다.
- **reader는 여럿이 동시에 해도 가능하다.** 
  - 이는 읽기만 하고 데이터를 변경하지 않기 때문이다.

- **solution**
  - Writer가 DB에 접근 허가를 아직 얻지 못한 상태에서는 모든 대기 중인 Reader들을 다 DB에 접근하게 해준다.
    - Writer는 대기 중인 Reader가 하나도 없을 때 DB 접근이 허용된다.
    - 그런데 만약 reader가 계속 나타나면? => **writer의 starvation 문제 발생**
  - Writer가 DB에 접근 중이면 Reader들은 접근이 금지된다.
    - Writer가 DB에서 빠져나가야만 Reader의 접근이 허용된다.
    - 똑같이 writer가 계속 쓴다면  **reader는 starvation!**



```c
// shared data
int readcount = 0;	// 현재 DB에 접근 중인 reader의 수
DB db;

// synchronization variables
semaphore mutex = 1, db = 1;

// Writer
P(db);
...
writing DB is performed
...
V(db);

// Reader
P(mutex);	// readcount를 위한 mutex
readcount++;
if(readcount == 1)	// 만약 reader의 처음이라면 lock!
    P(db);
V(mutex);
...
reading DB is performed
...
P(mutex);
readcount--;
if(readcount==0)	// 모든 reader가 끝났다면 unlock!
    V(db);
V(mutex);
```



### Dining- Philosophers Problem

#### 식사하는 철학자 문제

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_ch6-2_img1.jpg?raw=true" alt="week4_ch6-2_img1.jpg" style="zoom: 67%;" />

가정은 다음과 같다.

다섯 명의 철학자가 원탁에 앉아있다. 이들은 식사를 하려고 하는데, 젓가락은 총 5개(쌍X)가 있다. 

그렇다면 이들은 본인의 왼쪽, 오른쪽에 위치한 젓가락을 집어야 한다. 

그런데, 

만약 모든 철학자가 본인의 왼쪽에 있는 젓가락을 집을 경우, 오른쪽에 있었던 젓가락은 이미 다른 철학자가 가지고 있다.

결국 그 누구도 젓가락을 한 쌍을 가질 수 없고 식사를 할 수 없는 것이다.

즉, 젓가락은 공유 자원이므로 semaphore로 해결한다.

```c
//Synchronization variables
semaphore chopstick[5]; // initially all values are 1, 놓으면 1, 잡으면 0

// Philosopher i
do{
    P(chopstick[i]);		// 왼쪽 젓가락을 집는다.
    P(chopstick[(i+1)%5]);	// 오른쪽 젓가락을 집는다.
    ...
    eat();
    ...
    V(chopstick[i]);
    V(chopstick[(i+1)%5]);
    ...
    think();
    ...
}while(1);
```

그러나 문제는 여전히 존재한다.

**만약 모든 철학자가 동시에 왼쪽 젓가락을 집으면 Deadlock이 생긴다.**

해결 방안

- 4명의 철학자만이 테이블에 동시에 앉을 수 있도록 한다.
  - 한 명은 젓가락 2개를 집을 수 있다. (비둘기 집의 원리)
- 젓가락을 2개 모두 집을 수 있을 때에만 젓가락을 집게 한다.
- 비대칭
  - 짝수 철학자는 왼쪽 젓가락부터, 홀수 철학자는 오른쪽부터 집도록 한다. => 같은 젓가락이므로 둘 중 한 명이 먼저 잡아야만 젓가락을 모두 잡을 수 있다.



```c
enum {thinking, hungry, eating} state[5];
semaphore self[5] = 0;	// 젓가락 두 개를 모두 집을 수 있는지 확인
semaphore mutex = 1;

//Philosopher i
do{
    pickup(i);
    eat();
    putdown(i);
    think();
}while(1);

void putdown(int i){
    P(mutex);
    state[i] = thinking;
    test((i+4)%5);	
    test((i+1)%5);
    V(mutex);
}

void pickup(int i){
    P(mutex);		// lock
    state[i] = hungry;	// 상태를 배고픔으로 바꾸어 차례임을 밝힌다.
    test(i);			// test(i)
    V(mutex);
    P(self[i]);	// 만약 test에서 젓가락을 얻지 못하면 이미 self값은 0이므로 계속 기다린다. 
}

void test(int i)[
    if(state[(i+4)%5] !=eating && state[i] == hungry && state[(i+1)%5] != eating){	// i번의 왼쪽, 오른쪽이 젓가락을 집고 있지 않고 배고플 때
        state[i]= eating;
        V(self[i]);	// i가 젓가락을 들고 있음.
    }
]
```

monitor를 이용한 코드를 보자.

```c
monitor dining_philosopher
{
    enum{thinking, hungry, eating} state[5];
    condition self[5];
    
    void pickup(int i){
        state[i] = hungry;
        test(i);
        if(state[i] != eating)
            self[i].wait();
    }
    
    void putdown(int i){
        state[i] = thinking;
        
        test((i+4) % 5); // if Left is waiting
        test((i+1) % 5);
    }
    
    void test(int i){
        if((state[(i+4)%5] != eating) && (state[i] ==hungry) && (state[(i+1)%5] != eating)){
            state[i] = eating;
            self[i].signal();
        }
    }
    
    void init(){
        for(int i=0;i<5;i++){
            state[i] = thinking;
        }
    }
}

Each philosopher{
    pickup(i);
    eat();
    putdown(i);
    think();
}while(1);
```

