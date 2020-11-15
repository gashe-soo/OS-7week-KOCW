# Week 5. Problem

- Q. 조건이 다음과 같을 때 Effective Access Time을 계산하시오.

```
3page level paging system일 때
Memory Access Time = 110ns
TLB Access Time = 10ns
TLB hit ratio = 0.95
```



- Q. Inverted page table을 사용하는 이유와 동작 방식, 장단점에 대해 서술하시오







### Answer

- EAT = 0.95 * (110+10) + 0.05 *( 10+ 110\*4) =  114 + 22.5 = 136.5
- Inverted page table
  - 사용 이유 : 프로세스마다 page table을 가지며, page table도 여러 entry를 가진다. 이는 많은 양의 물리 메모리를 소비해야 하기 때문에 공간 효율이 떨어진다.
  - 동작 방식 : 기존의 page 번호 뿐만 아니라 프로세스 id를 활용해 물리 메모리의 주소를 확인한다. 
    - <img src="https://github.com/gashe-soo/OS-7week-KOCW/blob/main/asset/week5_ch8_img5.jpg?raw=true" alt="week5_ch8_img5.jpg"  />
  - 장점 : 물리 메모리의 공간 낭비를 막을 수 있다.
  - 단점 : 최악의 경우 모든 page table을 검색해야 하므로 시간 효율 저하



