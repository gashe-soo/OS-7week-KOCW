# Week 4. Problem

1. 다음은 dining philosopher problem을 monitor로 풀어내는 코드이다. (A),(B),(C)에 들어가는 코드와 코드의 의미를 작성하시오.

```c
monitor dining_philosopher{    
    enum{thinking, hungry, eating} state[5];    
    condition self[5];        
    
 	void pickup(int i){        
        state[i] = hungry;        
        test(i);        
        // problem A!!
        (A);
    }        
    void putdown(int i){        
        state[i] = thinking;                
        test((i+4) % 5);     
        test((i+1) % 5);    
    }        
    void test(int i){        
        // problem B!!
        if((B)){            
            state[i] = eating;            
            self[i].signal();
        }    
    }        
    void init(){        
        for(int i=0;i<5;i++){            
            //problem C
            (C);        
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



2. 현재 시스템의 상태가 아래와 같다. 은행원 알고리즘을 사용하여 다음 각 상태가 불안전 상태인지 결정하라. 만일 상태가 안전하다면 safe sequence를 설명하라. 불안전하다면 이유를 설명하라.

<img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_problem.jpg?raw=true" alt="week4_problem.jpg"  />



1) Available = (0, 3, 0, 1)

2) Available = (1, 0, 0, 2)





<br/>

<br/><br/><br/>

### Answer

1. - ```c
     // (A)는 젓가락을 얻지 못한 경우을 뜻한다. 그러므로 기다려야 한다.
     if(state[i] != eating)     
         self[i].wait();
     
     //(B)는 본인의 양쪽이 먹지 않고, 본인의 상태가 배고픈지를 확인한다 => 본인의 차례인지 확인한다.
     (state[(i+4)%5] != eating) && (state[i] == hungry) && (state[(i+1)%5] != eating)
     
     // (C) 초기화 코드로 철학자는 thinking이 기본이다.
     state[i] = thinking;
     ```

     

2. 필요한 자원은 아래와 같이 변할 것이다. 

   <img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week4_problem_answer.jpg?raw=true" alt="week4_problem_answer.jpg"  />

   1) 

   - (0,3,0,1) 일 때 P0,P1는 불가, P2는 가능하다. P2는 사용하고 자원 반납. (0,3,0,1) + (3,1,2,1)
   - (3,4,2,2) 일 때 P3,P4불가. 다시 P0도 불가, P1 가능 => (3,4,2,2) +(2,2,1,0) = (5,6,3,2)
   - (5,6,3,2) 일 때 P3 가능 => (5,6,3,2)+(0,5,1,0) = (5,11,4,2)
   - (5,11,4,2)일 때 P4불가 P0 불가. 즉 불안전한 상태이다. 
   - 왜냐하면 자원 D는 요청보다 부족하기 때문이다.

   2) (1,0,0,2)

   - (1,0,0,2) 일 때, P0 불가, P1 가능 => (1,0,0,2) + (2,2,1,0) = (3,2,1,2)
   - (3,2,1,2) 일 때, P2 가능  => (3,2,1,2) +(3,1,2,1) = (6,3,3,3)
   - (6,3,3,3) 일 때 P3 가능 => (6,3,3,3) + (0,5,1,0) = (6,8,4,3) 
   - (6,8,4,3) 일 때 P4 가능 => (6,8,4,3) + (4,2,1,2) = (10,10,5,5)
   - 마지막으로 P0도 가능하다.
   - 따라서 안전하며 순서는  <P1,P2,P3,P4,P0>이다.



