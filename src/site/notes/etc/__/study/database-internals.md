---
{"dg-publish":true,"permalink":"/etc/__/study/database-internals/","tags":["database"],"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---



### skiplist(스킵리스트)


- 검색,삽입,삭제에 대해 시간복잡도 O(log n)를 갖는, 정렬된 데이터를 갖는 확률적 자료구조 ([wikipedia](https://en.wikipedia.org/wiki/Skip_list))
- ![](https://i.imgur.com/o1G0sEG.png)
- 링크드리스트의 일종
- **지하철 노선도의 급행열차**


### latch(래치)


- 래치는 동시 액세스를 제어하고 데이터 일관성을 보장하기 위해 사용하는 동기화(synchronization) 메커니즘
- DBMS 는 일반적으로 여러 트랜잭션(*또는 스레드*)의 동시 액세스로부터 데이터 구조(*또는 버퍼*)와 같은 공유 리소스를 보호하기 위해 래치를 활용
- ![](https://i.imgur.com/YEyKJ3I.png)
- 대부분의 아티클에서는 Lock 과 비교함
    - 비교내용 (표)
    - ![|700](https://i.imgur.com/rbsnme7.png)


### cas(compare and swap, 비교연산)


- 동기화를 위해 멀티스레딩에서 사용되는 원자적 명령 in CS ([참고](https://en.wikipedia.org/wiki/Compare-and-swap))
- 예시
    - 여러 스레드(A, B)가 CAS(비교 및 스왑) 알고리즘을 사용해 동일한 변수를 동시에 업데이트하려고 시도한다 가정해보자. 한 스레드(A)가 승리하여 변수 값을 업데이트하고 나머지(B)는 업데이트에 실패한다. 하지만 패한 스레드(B)는 정지로 인한 불이익이 없다. 실패한 스레드(B and more..)들은 자유롭게 연산을 다시 시도하거나, 아무 동작도 안하면 된다.
- pseudo code
    ```
    function cas(p: pointer to int, old: int, new: int) is
        if *p ≠ old
            return false
    
        *p ← new
    
        return true
    ```
- [java example](https://www.geeksforgeeks.org/java-program-to-implement-cas-compare-and-swap-algorithm)
    - java5 이전에는 `syncronized` 블록을 사용해야했지만,
    - java5 이후에는 atomic 패키지를 제공하며 원자성을 보장한다.


### 


### reference


- skiplist: https://bravenamme.github.io/2021/01/21/tree-skiplist
- latch
    - https://stackoverflow.com/a/42464336
    - https://www.baeldung.com/cs/dbms-synchronization-lock-vs-latch
- cas
    - https://www.geeksforgeeks.org/java-program-to-implement-cas-compare-and-swap-algorithm
    - https://g.co/kgs/dFA7pa