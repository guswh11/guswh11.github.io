---
layout: post
title: "스레드풀(ThreadPool) 개념 및 사용법"
categories: [java]
comments: true
tags:
  - Java
  - Thread
  - ThreadPool
---
## 스레드풀 
스레드풀(ThreadPool): 작업 처리에 사용되는 스레드를 제한된 개수만큼 정해 놓고 작업 큐에 들어오는 작업들을 하나씩 스레드가 맡아 처리

**사용 이유**
- 프로그램의 성능 저하를 방지
    - 스레드를 매번 생성/수거하는데 따르는 부담은 애플리케이션의 성능을 저하시킴
- 다수의 사용자 요청을 처리 

![threadpool](thread-pool.jpg)

**장점**
- 병렬 작업의 폭증으로 인한 스레드의 폭증을 막음
    - 작업 처리 요청이 폭증되어도 스레드의 전체 개수가 늘어나지 않음
    - 애플리케이션의 급격한 성능 저하를 막음
- **스레드를 생성/수거하는데 드는 비용이 들지 않음**
    - 처음 생성하는 비용은 들지만, 이전의 스레드를 재사용함으로써 시스템자원을 아낌
- 작업 요청 시 이미 스레드가 대기중인 상태여서 작업을 실행하는 데 딜레이가 발생하지 않음

**단점**
- 메모리 낭비의 가능성
    - 스레드를 너무 많이 생성해 두었다가 사용하지 않는 경우

### 스레드풀 생성 및 종료
#### 스레드풀 생성
ExecutorService 구현 객체(스레드풀)는 Executors 클래스의 다음 두 가지 메서드 중 하나를 이용해서 생성 가능하다.

|메서드명(매개 변수)|초기 스레드 수|코어 스레드 수|최대 스레드 수|
|--|--|--|--|
|newCachedThreadPool()|0|0|Integer.MAX_VALUE|
|newFixedThreadPool(int nThreads)|0|nThreads|nThreads|

- 초기 스레드 수: ExecutorService 객체가 생성될 떄 기본적으로 생성되는 스레드 수
- 코어 스레드 수: 스레드 수가 증가된 후 사용하지 않는 스레드를 스레드풀에서 제거할 때 최소한 유지해야 할 스레드 수
    - 나중에 스레드 수를 다시 늘려야 할 때 생길 부하를 줄일 수 있다<br>(이미 몇개는 만들어져 있으므로)
- 최대 스레드 수: 스레드풀에서 관리하는 최대 스레드 수

ExecutorService 구현 객체는 아래와 같이 얻을 수 있다.

```java
ExecutorService executorService = Executors.newCachedThreadPool();
```

다음은 CPU 코어의 수만큼 최대 스레드를 사용하는 스레드풀을 생성한다. 
```java
ExecutorService executorService = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors()
);
```

또는 다음과 같이 직접 ThreadPoolExecutor 객체를 생성할 수 있다. 

```java
ExecutorService threadPool = new ThreadPoolExecutor(
    3, // 코어 스레드 수
    100, // 최대 스레드 수
    120L, // 놀고 있는 시간
    TimeUnit.SECONDS, // 놀고 있는 시간 단위
    new SynchronousQueue<Runnable>() // 작업 큐
);
```

#### 스레드풀 종료
스레드풀의 스레드는 기본적으로 데몬 스레드가 아니기 때문에, main 스레드가 종료되더라도 계속 실행 상태로 남아있다.<br>애플리케이션을 종료하려면 스레드풀을 종료시켜 스레드들이 종료 상태가 되도록 처리해주어야 한다. 

ExecutorService는 종료와 관련해서 다음 세 메서드를 제공한다. 

|리턴 타입|메서드명(매개 변수)|설명|
|--|--|--|
|void|shutdown()|현재 처리 중인 작업뿐만 아니라 작업 큐에 대기하고 있는 모든 작업을 처리한 후에 스레드풀을 종료시킨다.|
|List<Runnable>|shutdownNow()|현재 작업 중인 스레드를 interrupt해서 작업 중지를 시도하고 스레드풀을 종료시킨다.<br>리턴값은 작업 큐에 있는 미처리된 작업(Runnable)의 목록이다.|
|boolean|awaitTermination(long timeout, TimeUnit unit)|shutdown() 메서드 호출 이후, 모든 작업 처리를 timeout 시간 내에 완료하면 true를 리턴하고, 완료하지 못하면 작업 처리 중인 스레드를 interrupt하고 false를 리턴한다.|

일반적으로 남아있는 작업을 마무리하고 스레드풀을 종료할 때는 shutdown()을 호출하고, 남아있는 작업과 상관없이 강제로 종료할 때는 shutdownNow()를 호출한다. 

종료하지 않을 경우 메모리 릭이 발생할 수 있기 때문에, 종료는 필수적이다. 

### 작업 생성과 처리 요청
#### 작업 생성
하나의 작업은 Runnable 또는 Callable 구현 클래스로 표현한다. 차이점은 작업 처리 완료 후 리턴값이 있느냐 없느냐이다. 

**Runnable 구현 클래스**

```java
Runnable task = new Runnable() {
    @Override 
    public void run() {
        // 작업 내용 
    }
}
```

**Callable 구현 클래스**

```java
Callable<T> task = new Callable<T>() {
    @Override 
    public T call() throws Exception {
        // 작업 내용 
        return T; 
    }
}
```

스레드풀의 스레드는 작업 큐에서 Runnable 또는 Callable 객체를 가져와 run()과 call() 메서드를 실행한다. 

#### 작업 처리 요청 
작업 처리 요청: ExecutorService 작업 큐에 Runnable 또는 Callable 객체를 넣는 행위 

ExecutorService는 작업 처리 요청을 위해 두 가지 종류의 메서드를 제공한다. 

|리턴 타입|메서드명(매개 변수)|설명|
|-|-|-|
|void|execute(Runnable command)|Runnable을 작업 큐에 저장<br>작업 처리 결과를 받지 못함(리턴값 없음)|
|Future<?><br>Future<V><br>Future<V>|submit(Runnable task)<br>submit(Runnable task, V result)<br>submit(Callable<V> task)|Runnable 또는 Callable을 작업 큐에 저장<br>리턴된 Future을 통해 작업 처리 결과를 얻을 수 있음|

**execute()**
- 작업 처리 결과를 받지 못함
- 작업 처리 도중 예외가 발생하면 스레드가 종료되고 해당 스레드는 스레드풀에서 제거됨
    - 다른 작업 처리를 위해 새로운 스레드 생성

**submit()**
- Future 리턴을 통해 작업 처리 결과를 받음
- 작업 처리 도중 예외가 발생하더라도 스레드는 종료되지 않고 다음 작업을 위해 재사용됨 
    - 스레드 생성 오버헤더를 줄여줌

### 블로킹 방식의 작업 완료 통보
Executorservice의 submit() 메서드는 매개값으로 준 Runnable 또는 Callable 작업을 스레드 풀의 작업 큐에 저장하고 즉시 Future 객체를 리턴한다. 

|리턴 타입|메서드명(매개 변수)|설명|
|-|-|-|
|Future<?><br>Future<V><br>Future<V>|submit(Runnable task)<br>submit(Runnable task, V result)<br>submit(Callable<V> task)|Runnable 또는 Callable을 작업 큐에 저장<br>리턴된 Future을 통해 작업 처리 결과를 얻을 수 있음|

Future: 지연 완료(pending completion) 객체
- 작업이 완료될 때까지 기다렸다가(지연했다가=블로킹되었다가) 최종 결과를 얻는 데 사용됨

Future의 get() 메서드를 호출하면 스레드가 작업을 완료할 때까지 블로킹되었다가 작업을 완료하면 처리 결과를 리턴한다 -> <u>블로킹을 사용하는 작업 완료 통보 방식</u>

다음은 Future가 가지고 있는 get() 메서드에 대한 표이다. 

|리턴 타입|메서드명(매개 변수)|설명|
|-|-|-|
|V|get()|작업이 완료될 때까지 블로킹되었다가 처리 결과 V를 리턴|
|V|get(long timeout, TimeUnit unit)|timeout 시간 전에 작업이 완료되면 결과 V를 리턴하지만, 작업이 완료되지 않으면 TimeoutException을 발생시킴|

다음은 세 가지 submit() 메서드별로 Future의 get() 메서드가 리턴하는 값이 무엇인지 보여준다. 

|메서드|작업 처리 완료 후 리턴 타입|작업 처리 도중 예외 발생|
|-|-|-|
|submit(Runnable task)|future.get()->null|future.get()->예외 발생|
|submit(Runnable task, Integer result)|future.get()->int 타입 값|future.get()->예외 발생|
|submit(Callable<String> task)|future.get()->String|future.get()->예외 발생|

Future를 이용한 블로킹 방식의 작업 완료 통보에서 작업을 처리하는 스레드가 작업을 완료하기 전까지는 get() 메서드가 블로킹되어 다른 코드를 실행할 수 없다.<br>그렇기 떄문에 get() 메서드를 호출하는 스레드는 새로운 스레드이거나 스레드풀의 또 다른 스레드가 되어야 한다. 

- 새로운 스레드를 생성해서 호출

    ```java
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                future.get();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }).start();
    ```

- 스레드풀의 스레드가 호출

    ```java
    executorService.submit(new Runnable() {
        @Override
        public void run() {
            try {
                future.get();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    });
    ```

Future 객체는 get() 메서드 이외에도 다음과 같은 메서드를 제공한다. 
|리턴 타입|메서드명(매개 변수)|설명|
|-|-|-|
|boolean|cancel(boolean mayInterruptIfRunning)|작업 처리가 진행 중일 경우 취소시킴|
|boolean|isCancelled()|작업이 취소되었는지 여부|
|boolean|isDone()|작업 처리가 완료되었는지 여부|

- cancel(): 작업을 취소하고 싶을 경우 호출
    - 작업이 시작되기 전이라면 mayInterruptIfRunning 매개값과는 상관없이 작업 취소 후 true를 리턴
    - 작업이 진행 중이라면 mayInterruptIfRunning 매개값이 true일 경우에만 작업 스레드를 interrupt 함
    - 작업이 이미 완료되었거나 취소할 수 없을 경우, false를 리턴
- isCancelled()
    - 작업이 완료되기 전에 취소되었을 경우에만 true를 리턴
- isDone()
    - 작업이 정상적/예외/취소 등 어떤 이유에서건 작업이 완료되었다면 true를 리턴 

#### 리턴값이 없는 작업 완료 통보
리턴값이 없는 작업일 경우는 Runnable 객체로 생성한다.

```java
Runnble task = new Runnable() {
    @Override
    public void run() {
        // 작업 내용
    }
};
```

작업 처리 요청은 submit(Runnable task) 메서드를 이용한다.<br>리턴값인 Future 객체는 스레드가 작업 처리를 정상적으로 완료했는지, 작업 처리 도중에 예외가 발생했는지 확인하기 위해서이다. 

```java
Future future = executorService.submit(task); 
```

작업 처리가 정상적으로 완료되었다면 Future의 get() 메서드는 null을 리턴한다.<br>아래는 예외 처리 방법이다. 

```java
try{
    future.get();
} catch (InterruptedException e) {
    // 작업 처리 도중 스레드가 interrupt 될 경우
} catch (ExecutionException e) {
    // 작업 처리 도중 예외가 발생된 경우
}
```

#### 리턴값이 있는 작업 완료 통보
스레드가 작업을 완료한 후에 애플리케이션이 처리 결과를 얻어야 한다면 작업 객체를 Callable로 생성한다.<br>제네릭 타입 파라미터 T는 call() 메서드가 리턴하는 타입이다. 

```java
Callable<T> task = new Callable<T>() {
    @Override
    public T call() throws Exception {
        // 작업 내용
        return T; 
    }
};
```

작업 처리 요청은 ExecutorService의 submit() 메서드를 호출한다.<br>submit() 메서드는 작업 큐에 Callable 객체를 저장하고 즉시 Future<T>를 리턴한다.

```java
Future<T> future = executorService.submit(task);
```

스레드가 Callable 객체의 call() 메서드를 모두 실행하고 T 타입의 값을 리턴하면, Future<T>의 get() 메서드는 블로킹이 해제되고 T 타입의 값을 리턴한다. 

```java
try{
    T result = future.get();
} catch (InterruptedException e) {
    // 작업 처리 도중 스레드가 interrupt 될 경우
} catch (ExecutionException e) {
    // 작업 처리 도중 예외가 발생된 경우
}
```

#### 작업 처리 결과를 외부 객체에 저장
외부 Result 객체는 대개 공유 객체가 되어, 두 개 이상의 스레드 작업을 취합할 목적으로 이용된다. 

![result object](thread-result-object.png)

`submit(Runnable task, V result)` 메서드를 호출하면 즉시 Future<V>가 리턴되는데, Future의 get() 메서드를 호출하면 스레드가 작업을 완료할 때까지 블로킹되었다가 작업을 완료하면 V 타입 객체를 리턴한다. 

```java
Result result = ...;
Runnable task = new Task(result); 
Future<Result> future = executorService.submit(task, result); 
result = future.get();
```

외부 Result 객체는 생성자를 통해 주입받는다. 

```java
class Task implements Runnable {
    Result result;
    Task(Result result) { this.result = result; }
    @Override
    public void run() {
        // 작업 코드
        // 처리 결과를 result 저장
    }
}
```

#### 작업 완료 순으로 통보
`CompletionService`를 이용해 작업 처리가 완료된 것부터 결과를 통보받을 수 있다.<br>CompletionService는 처리 완료된 작업을 가져오는 poll()과 take() 메서드를 제공한다. 

|리턴 타입|메서드명(매개 변수)|설명|
|-|-|-|
|Future<V>|poll()|완료된 작업의 Future를 가져옴<br>완료된 작업이 없다면 즉시 null을 리턴|
|Future<V>|poll(long timeout, TimeUnit unit)|완료된 작업의 Future를 가져옴<br>완료된 작업이 없다면 timeout까지 블로킹됨|
|Future<V>|take()|완료된 작업의 Future를 가져옴<br>완료된 작업이 없다면 생길 때까지 블로킹됨|
|Future<V>|submit(Callable<V> task)|스레드풀에 Callable 작업 처리 요청|
|Future<V>|submit(Runnable task, V result)|스레드풀에 Runnable 작업 처리 요청|

CompletionService 구현 클래스는 ExecutorCompletionService<V>이다.<br>객체를 생성할 때 생성자 매개값으로 ExecutorService를 제공하면 된다. 

```java
ExecutorService executorService = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors()
);
CompletionService<V> completionService = new ExecutorCompletionService<V>(
    executorService
);
```

poll()과 take() 메서드를 이용해서 처리 완료된 작업의 Future를 얻으려면 CompletionService의 submit() 메서드로 작업 처리 요청을 해야 한다.

```java
completionService.submit(Callable<V> task); 
completionService.submit(Runnable task, V result); 
```

take() 메서드가 리턴하는 완료된 작업은 submit()으로 처리 요청한 작업의 순서가 아니다.<br>작업의 내용에 따라서 먼저 요청한 작업이 나중에 완료될 수도 있기 때문이다. 

### 콜백 방식의 작업 완료 통보
콜백: 애플리케이션이 스레드에게 작업 처리를 요청한 후, 스레드가 작업을 완료하면 특정 메서드(<u>콜백 메서드</u>)를 자동 실행하는 기법 

![blocking vs callback method](blocking-vs-callback-method.png)

- 블로킹 방식: 작업 처리를 요청한 후 작업이 완료될 때까지 블로킹됨
- 콜백 방식: 작업 처리를 요청한 후 결과를 기다릴 필요 없이 다른 기능 수행 가능
    - 작업 처리가 완료되면 자동으로 콜백 메서드가 실행되어 결과를 알 수 있기 때문

ExecutorService는 콜백을 위한 별도의 기능을 제공하지 않지만, Runnable 구현 클래스를 작성할 때 콜백 기능을 구현할 수 있다. 

콜백 메서드를 가진 클래스는 직접 정의하거나 `java.nio.channels.CompletionHandler`를 이용한다. 이 인터페이스는 비동기 통신에서 콜백 객체를 만들 때 사용된다. 

```java
CompletionHandler<V, A> callback = new CompletionHandler<V, A>() {
    @Override
    public void completed(V result, A attachment) {

    }

    @Override
    public void failed(Throwable exc, A attachment) {

    }
};
```

- completed(): 작업을 정상 처리 완료했을 때 호출되는 콜백 메서드
- failed(): 작업 처리 도중 예외가 발생했을 때 호출되는 콜백 메서드

아래는 작업 처리 결과에 따라 콜백 메서드를 호출하는 Runnable 객체이다. 

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        try{
            // 작업 처리
            V result = ...;
            callback.completed(result, null); 
        } catch(Exception e) {
            callback.failed(e, null); 
        }
    }
};
```

#### Reference 
<https://www.wrapuppro.com/programing/view/jAuG3VNBCbGnQWU>