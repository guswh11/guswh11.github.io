---
layout: post
title: "프로세스 동기화"
categories: [OS]
comments: true
tags:
  - OS
  - Process
  - Synchronization
---

# 프로세스 동기화(Process Synchronization)
프로세스들이 공유 데이터에 동시 접근하는 것은 데이터의 일관성을 해칠 수 있다.<br>
프로세스 동기화는 프로세스들의 질서있는 실행과 데이터의 일관성을 유지하는 방법에 대해 다룬다. 

**경쟁 조건(Race Condition)**
- 여러 프로세스가 공유 데이터를 동시에 조작할 때, 실행의 특정 순서에 따라 결과가 달라지는 상황
    - 연산의 결과를 신뢰할 수 없게 만든다. 
- 공유 데이터 조작을 원자적 연산으로 처리하면 경쟁 조건은 발생하지 않는다.

> 원자적(atomic): 수행 도중 중단될 수 없는 동작 단위

## 임계 영역 문제(The Critical-Section Problem)
**임계 영역(Critical Section)**

![critical section](critical-section.png)

임계 영역이란 각 프로세스에서 공유 데이터에 접근하는 프로그램 코드 부분을 의미한다.<br>
임계 영역은 다른 프로세스와 공유하는 변수를 변경하거나, 테이블을 갱신하거나 파일을 쓰는 등의 작업을 실행한다.<br>

경쟁 조건 상황을 방지하기 위해, 다음 세 가지 요구조건을 충족해야 한다. 
- **상호 배제(Mutual exclusion)**: 한 프로세스가 자신의 임계 영역에서 실행할 동안, 다른 프로세스는 자신의 임계 영역에서 실행될 수 없다. 
- **진행(Progress)**: 어떤 프로세스도 임계 영역에서 실행되고 있지 않다면, 임계 영역으로 진입하려는 프로세스들 중 하나는 유한한 시간 내에 진입할 수 있어야 한다. 
- **유한 대기(Bounded waiting)**: 한 프로세스가 임계 영역에 대한 진입을 요청하면, 다른 프로세스의 임계 영역 진입은 유한한 횟수로 제한되어야 한다. 
- 임계 영역에 대한 진입 요청 후 무한히 기다리지 않음

## 동기화 방법
### Lock 사용
임계 영역 문제에 대한 가장 기본적인 해결 방법은 락(lock)을 사용하는 것이다.  
- 프로세스는 임계 영역에 진입하기 전에 반드시 락을 획득해야 한다. 
- 임계 영역을 나올 때는 락을 방출한다. 

```c++
do {
    // 락 획득 

    임계 영역 

    //락 방출

    나머지 영역
} while(true)
```

락 획득 도중 인터럽트가 걸리는 문제 상황이 발생할 수 있다. 

### 해결 방법 
**HW solution**
더 이상 쪼개지지 않는 원자적 하드웨어 명령어를 이용하는 방법이다.
- TestAndSet (TAS) instruction

**OS supported SW solution**
- Spinlock
- Semaphore
- Eventcount/sequencer

**Language-Level solution**
- Monitor

### TestAndSet(TAS) instruction
- 락을 원자적으로 처리하는 하드웨어 명령어
    - 중간에 인터럽트 되지 않는 명령어(non-preemptive)
    - TestAndSet() : 한 워드의 내용을 확인하고 수정하는 연산을 원자적으로 처리

        ```c++ 
        Boolean TestAndSet(boolean *target) {
            boolean rv = *target; 
            *target = true; 
            return rv;
        }
        ```

        ```c++
        do {
            while (TestAndSet(&lock))
            ; // do nothing

            // critical section

            lock = FALSE; 

            //remainder section

        } while (TRUE);
        ```

    - Swap() : 두 워드에 내용을 서로 교환하는 연산을 원자적으로 처리

- busy waiting 존재 
- 두 명령어 모두 제한된 대기를 만족시키지 않음 
    - 제한된 대기를 보장하는 TestAndSet 알고리즘이 존재하지만, 구현이 복잡해 지원되지 않음
    
### 세마포어(Semaphore)
세마포어 S는 정수 변수로, 초기화를 제외하고는 원자적 계산 wait(), signal()로만 접근이 가능하다.

- wait(): 락 획득

    ```c++
    wait(S) {
        while (S<=0)
        ; // 아무런 작업을 하지 않음 
        S--; 
    }
    ```

- signal(): 락 해제

    ```c++
    signal(S) {
        S++; 
    }
    ```

- 세마포어 값을 변경하는 연산은 원자적이다. 
    - 한 스레드가 세마포어 값을 변경하면 다른 어떤 스레드도 동시에 동일한 세마포어 값을 변경할 수 없다.
- 세마포어는 가용한 자원의 개수로 초기화됨

#### 사용법
- 카운팅(counting) 세마포어: 값에 제한이 없음
    - 유한한 개수를 가진 자원에 대한 접근 제어에 사용
- 이진(binary) 세마포어: 0과 1 사이의 값만 가짐
    - <u>뮤텍스(mutex) 락</u>이라고 부름

다중 프로세서 시스템에서 n개의 프로세서들은 뮤텍스라는 세마포어를 공유하고, 초기값은 1이다.<br>
세마포어 값이 0 -> 모든 자원이 사용중임 

```c++
do {
    wait(mutex);
    
    // critical section

    signal(mutex);

    // remainder section
} while(TRUE); 
```

#### 구현
위와 같은 세마포어의 구현은  **바쁜 대기**(busy waiting)를 요구한다.<br>
**스핀락(spinlock)**: 한 프로세스가 자신의 임계 영역에 있으면 다른 프로세스들은 진입 코드를 계속 반복 실행해야 함

**장점**
- 락이 짧은 시간 동안만 소유될 경우 유용함 
- 멀티프로세서/멀티코어 시스템에서 채택됨
    - 동시에 프로세스/스레드가 실행 가능해야 busy waiting 도중 락이 풀리는 경우를 기대 가능 
    - 문맥 교환 비용을 절약

**단점**
- CPU 시간을 낭비하게 됨 

**바쁜 대기 해결 방법**
세마포어 값이 양수가 아닌 경우 프로세스는 대기하는 것이 아니라, 자신을 봉쇄한다. 
- 프로세스는 세마포어에 연관된 대기 큐에 넣어짐 
- 프로세스를 대기 상태로 전환
- 봉쇄된 프로세스는 `wakeup()` 연산에 의해 재시작됨 
    - 프로세스를 대기에서 준비 완료 상태로 변경

이 경우, 세마포어는 구조체로 정의된다. 

```c
typedef struct {
    int value; 
    struct process *list;
} semaphore; 
```

프로세스가 세마포어를 기다려야 한다면, 이 프로세스는 세마포어의 프로세스 리스트에 추가된다.  signal() 연산은 프로세스 리스트에서 한 프로세스를 꺼내서 그 프로세스를 깨워준다. 

```c
void wait(semaphore *S) {
    S->value--; 
    if (S->value < 0){
        프로세스를 s->list 안에 넣음;
        block(); 
    }
}

void signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) {
        S->list로부터 프로세스 P를 꺼냄;
        wakeup(P);
    }
}
```

이 정의에서 `S->value`의 값은 음수일 수 있다. 음수일 때, 그 절댓값은 대기하고 있는 프로세스들의 수이다. 

#### 교착상태와 기아(Deadlock and Starvation)

![deadlock](deadlock.gif)

#### 우선순위 역전(Priority Inversion)
우선순위 역전: 높은 우선순위의 프로세스가 낮은 우선순위의 프로세스로 인해 수행이 block된 상태

**발생 상황**(우선순위가 높은 순서대로 A, B, C)
- C가 먼저 세마포어를 요청해 획득한 상태. 실행 도중 A가 세마포어를 요청하지만, C가 세마포어를 반환할 때까지 block 됨.
-> 우선순위가 높더라도 세마포어가 없으면 공유자원을 사용하지 못함(제한된 우선순위 역전 상황)
-  C가 먼저 세마포어를 요청해 획득한 상태. 실행 도중 A가 세마포어를 요청하지만, C가 세마포어를 획득한 상태이므로 block 됨. C보다 더 높은 우선순위의 프로세스들(B 수준)이 실행되고, C는 세마포어를 반납하지 못하는 상태가 지속됨. 

**우선순위 상속 프로토콜(priority-inheritance protocol)**
더 높은 우선순위 프로세스가 필요로 하는 자원에 접근하는 모든 프로세스들은 해당 자원을 사용하는 동안 더 높은 우선순위를 상속받는다. 자원 사용이 끝나면 원래 우선순위로 되돌아간다. 

위 예에서 우선순위 상속 프로토콜에 따르면 C가 A의 우선순위를 상속받고, B가 선점해 실행되는 것을 막는다. 

### 모니터(Monitors) - Java, c# 찾을것 
세마포어는 특정 실행 순서로 진행되었을 때 타이밍 오류를 야기할 수 있다는 단점이 있다. 세마포어는 동기화 함수의 제약 조건을 고려해야 하는 반면, 모니터는 프로시져를 호출하여 간단히 해결할 수 있다.

모니터는 자바 프로그램에서 스레드를 동기화하는 방법으로써 활용도가 높다. 

**구조**

![monitor architecture](monitor-architecture.png)

- 공유자원과 공유자원에 대한 접근함수(임계구역)가 존재함
-  두 개의 큐
    - **배타 동기 큐(Mutual Exclusion Queue)**: 하나의 스레드만 공유자원에 접근하도록 함
        - 특정 스레드가 임계구역에 있으면 다른 스레드는 배타 동기 큐에서 대기함 
    - **조건 동기 큐(Conditional Synchronization Queue)**: 진입 스레드가 block되고 새 스레드를 진입 가능하게 하는 공간
        - 공유자원을 사용중인 스레드가 `wait()`을 통해 들어감
        - `notify()`를 통해 조건 동기 큐의 스레드를 깨울 수 있음 

배타 동기는 `synchronized` 키워드를 사용해서 지정할 수 있다. 아래 코드에서 func1 메서드와 func2 메서드는 **같은 임계구역을 갖는다**. 

```java
class C {
    private int value;
    synchronized void func1() {
        ...
    }

    synchronized void func2() {
        ...
    }
}
```

조건 동기는 `wait()`, `notify()`, `notifyAll()` 메서드를 사용한다. 
- wait(): 호출한 쓰레드를 조건동기 큐에 삽입한다.

    ![wait()](monitor-wait.png)

- notify(): 조건동기 큐에 있는 하나의 쓰레드를 깨워준다.
    
    ![notify()](monitor-notify.png)
    
    - 현재 수행중인 스레드의 synchronized 블록이 끝나면 대기중이던 스레드가 실행된다. 
- notifyAll(): 조건동기 큐에 있는 모든 쓰레드를 깨워준다.

모니터 역시 notify(), wait() 메서드를 이용해 스레드의 실행 순서를 조정할 수 있다.<br>프로세스 1과 프로세스 2에 대해 프로세스 1을 먼저 실행시키고 싶은 경우
1. 프로세스 2의 실행 코드 전에 wait()을 호출
    - 프로세스 2는 임계구역에서 코드를 실행시키기 전에 조건 동기 큐에 들어가게 됨
    - 실행 권한이 프로세스 1로 넘어감
2. 프로세스 1의 실행이 끝나고 notify()를 통해 프로세스 2를 깨움


#### Reference 
<https://jhnyang.tistory.com/36?category=815411><br>
<https://m.blog.naver.com/PostView.nhn?blogId=three_letter&logNo=220379691616&proxyReferer=https:%2F%2Fwww.google.com%2F>