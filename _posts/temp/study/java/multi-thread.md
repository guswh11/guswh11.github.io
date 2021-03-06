# 멀티 스레드
## 멀티 스레드 개념 
멀티 스레드는 하나의 프로세스가 두 가지 이상의 작업을 처리할 수 있도록 한다. 
ex) 메신저-채팅 기능을 제공하면서 파일 전송 기능도 수행

**멀티 프로세스**: 애플리케이션 단위의 멀티 태스킹
- 각자의 메모리를 가짐, 서로 독립적
    - 하나의 프로세스에서 오류가 발생해도 다른 프로세스에 영향 x

**멀티 스레드**: 애플리케이션 내부에서의 멀티 태스킹
- 프로세스 내부에 생성됨
    - 하나의 스레드가 예외를 발생시키면 프로세스 자체가 종료될 수 있음
        - 다른 스레드에게 영향을 미침

### 메인 스레드 
모든 자바 애플리케이션은 메인 스레드(main thread)가 main() 메서드를 실행하면서 시작된다. 

메인 스레드는 필요에 따라 작업 스레드들을 만들어 병렬적으로 코드를 실행할 수 있다. 

![single/multi thread java application](java-thread.png)

**싱글 스레드 애플리케이션**
- 메인 스레드가 종료하면 프로세스도 종료

**멀티 스레드 애플리케이션**
- 실행 중인 스레드가 하나라도 있다면 프로세스는 종료되지 않음
    - 메인 스레드가 작업 스레드보다 먼저 종료되더라도 작업 스레드가 아직 실행 중이라면 프로세스 종료 x

## 작업 스레드 생성과 실행
### Thread 클래스로부터 직접 생성 

### Thread 하위 클래스로부터 생성 

### 스레드의 이름

## 스레드 우선순위 
멀티 스레드는 동시성(Concurrency) 또는 병렬성(Parallelism)으로 실행된다.<br>
동시성: 멀티 작업을 위해 <u>하나의 코어에서 멀티 스레드가 번갈아가며 실행</u>하는 성질
병렬성: 멀티 작업을 위해 <u>멀티 코어에서 개별 스레드를 동시에 실행</u>하는 성질 

![concurrency/parallelism](concurrency-parallelism.png)

스레드의 개수가 코어의 수보다 많을 경우, 스레드를 어떤 순서에 의해 동시성으로 실행할 것인가를 결정하는 것을 스케줄링이라고 한다.<br>
스레드 스케줄링에 의해 스레드들은 아주 짧은 시간에 번갈아가면서 각자의 run() 메서드를 조금씩 실행한다.

자바의 스레드 스케줄링은 우선순위(Priority) 방식과 순환 할당(Round-Robin) 방식을 사용한다. 

- **우선순위 방식**: 우선순위가 높은 스레드가 실행 상태를 더 많이 가지도록 스케줄링하는 것
    - 스레드 객체에 우선순위 번호를 부여 -> 개발자가 코드로 제어 가능

- **순환 할당 방식**: 시간 할당량(Time Slice)를 정해서 하나의 스레드를 정해진 시간만큼 실행하고, 다시 다른 스레드를 실행하는 방식
    - 자바 가상 기계에 의해서 정해짐 -> 코드로 제어 불가 

// 방법

## 동기화 메서드와 동기화 블록
### 공유 객체를 사용 시 주의점
멀티 스레드 프로그램에서는 스레드들이 객체를 공유해서 작업하는 경우가 있다.<br>
이 경우, 스레드 A가 사용하던 객체가 스레드 B에 의해 상태가 변경될 수 있기 떄문에 스레드 A가 의도했던 것과 다른 결과가 산출될 수 있다. 

### 동기화 메서드 및 동기화 블록
스레드가 사용 중인 객체를 다른 스레드가 변경할 수 없도록 하려면 스레드 작업이 끝날 때까지 객체에 잠금을 걸어서 다른 스레드가 사용할 수 없도록 해야 한다. 

임계 영역(critical section): 단 하나의 스레드만 실행할 수 있는 코드 영역
- 공유 자원의 독점을 보장하는 영역 
- 지정된 시간이 지난 후 종료됨
- 임계 영역을 지정하기 위해 동기화(synchronized) 메서드와 동기화 블록을 제공

메서드 선언에 `synchronized` 키워드를 붙여 동기화 메서드를 만들 수 있다. 스레드가 객체 내부의 동기화 메서드 또는 블록에 들어가면, 즉시 객체에 잠금을 걸어 다른 스레드가 임계 영역 코드를 실행하지 못하도록 한다. 

```java
public synchronized void method() {
    // 임계 영역 
}
```

- 메서드 전체 내용이 임계 영역
- 스레드가 동기화 메서드를 실행하는 즉시 객체에는 잠금이 일어남(객체를 사용하려는 다른 스레드는 대기하게 됨)
- 스레드가 동기화 메서드를 실행 종료하면 잠금 풀림

아래와 같이 동기화 블록을 통해 일부 내용만 임계 영역으로 만들 수 있다.

```java
public void method() {
    // 여러 스레드가 실행 가능 역역

    synchronized(공유객체) {
        // 임계 영역 
    }
}
```

## 스레드 상태 
- 실행 대기 상태: 스케줄링이 되지 않아서 실행을 기다리고 있는 상태
- 실행 상태: 실행 대기 상태에 있는 스레드 중에서 스레드 스케줄링으로 선택된 스레드가 CPU를 점유하고 run() 메서드를 실행
- 종료 상태: run() 메서드가 종료됨
- 일시 정지 상태: 스레드가 실행할 수 없는 상태 
    - 일시 정지 상태에서 실행 대기 상태로 가야 다시 실행 상태로 갈 수 있음 

![thread state](thread-state.png)

Thread 클래스의 `getState()` 메서드는 스레드의 상태에 따라 Thread.State 열거 상수를 리턴한다. 

|상태|열거 상수|설명|
|-|-|-|
|객체 생성|NEW|스레드 객체가 생성됨, 아직 start() 메서드가 호출되지 않은 상태|
|실행 대기|RUNNABLE|실행 상태로 언제든지 갈 수 있는 상태|
|일시 정지|WAITING|다른 스레드가 통지할 때까지 기다리는 상태|
|일시 정지|TIMED_WAITING|주어진 시간 동안 기다리는 상태|
|일시 정지|BLOCKED|사용하고자 하는 객체의 락이 풀릴때까지 기다리는 상태|
|종료|TERMINATED|실행을 마친 상태|

## 스레드 상태 제어

![thread state management](thread-state-management.png)

줄이 쳐진 메서드들은 deprecated 되었다. 

|메서드|설명|
|-|-|
|interrupt()|일시 정지 상태의 스레드에서 `InterruptedException` 예외를 발생시켜, 예외 처리 코드(catch)에서 실행 대기 상태로 가거나 종료 상태로 갈 수 있도록 함|
|notify()<br>notifyAll()|동기화 블록 내에서 wait() 메서드에 의해 일시 정지 상태에 있는 스레드를 실행 대기 상태로 만듦|
|sleep(long millis)<br>sleep(long millis, int nanos)|주어진 시간 동안 스레드를 일시 정지 상태로 만듦<br>주어진 시간이 지나면 자동적으로 실행 대기 상태가 된다.|
|join()<br>join(long millis)<br>join(long millis, int nanos)|join() 메서드를 호출한 스레드는 일시 정지 상태가 된다.<br>실행 대기 상태로 가려면 join() 메서드를 멤버로 가지는 스레드가 종료되거나, 매개값으로 주어진 시간이 지나야 한다.|
|wait()<br>wait(long millis)<br>wait(long millis, int nanos)|동기화 블록 내에서 스레드를 일시 정지 상태로 만든다. 주어진 시간이 지나면 자동적으로 실행 대기 상태가 된다. 시간이 주어지지 않으면 notify(), notifyAll() 메서드에 의해 실행 대기 상태로 갈 수 있다.|
|yield()|실행 중에 우선순위가 동일한 다른 스레드에게 실행을 양보하고 실행 대기 상태가 된다.|

// 사용 방법 새로 만들기 

## 데몬 스레드 
데몬(daemon) 스레드: 주 스레드의 작업을 돕는 보조적인 스레드
- 주 스레드가 종료되면 강제적으로 자동 종료 
    ex) 워드프로세서의 자동 저장, 미디어 플레이어의 동영상 및 음악 재생, 가비지 컬렉터 등

스레드를 데몬으로 만들기 위해서는 `setDaemon(true)`를 호출해주면 된다. 

```java
public static void main(String[] args){
    AutoSaveThread thread = new AutoSaveThread();
    thread.setDaemon(true); 
    thread.start();
    ...
}
```

`isDaemon()` 메서드를 통해 스레드가 데몬 스레드인지 아닌지 구별할 수 있다. 데몬 스레드일 경우 true를 리턴한다. 