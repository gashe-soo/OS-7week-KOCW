# Week 7. Problem



- Q. 다음 질문에 답하시오

  ```
  1) FAT는 (A) 할당 방법을 사용했다. A에 들어갈 말과 A의 장단점을 서술하시오.
  2) 1번의 A의 단점을 FAT는 어떻게 해결했는가?
  ```

<br><br><br>

- Q. Page Cache와 Buffer Cache를 합친 Unified Buffer Cache가 있다. 왜 두 개의 캐시를 합쳤는가?

<br>

<br><br><br><br>

<br><br><br>

### ANSWER

- A. 

```
1) FAT는 linked allocation을 사용했다. linked allocation은 외부 단편화가 발생하지 않는다. 
다만, Direct access가 불가하며, 포인터의 유실로 인한 reliability의 문제, 낮은 공간 효율성의 문제가 있다.
2) FAT는 pointer를 별도의 위치에 보관하여 reliability와 공간효율성 문제를 해결했다.
```



- A. I/O mapped I/O일 때는 buffer cache만을 이용하지만, memory mapped I/O일 경우 buffer cache, page cache에 동일한 내용을 적재하는 이중 캐시의 문제로 공간 효율성이 하락한다.