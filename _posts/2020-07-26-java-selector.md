---
layout: post
title: "Java-TCP 넌블로킹 채널과 셀렉터(Selector)"
categories: [java]
comments: true
tags:
  - Java
  - Selector
  - TCP
  - Non-blocking
---

### 넌블로킹 방식
블로킹 방식
- accept(), read()에서 블로킹됨 
- ServerSocketChannel과 연결된 SocketChannel당 하나의 스레드가 할당되어야 함
- 성능 문제 완화를 위해 스레드풀 사용

넌블로킹 방식 
- connect(), accept(), read(), write() 메서드에서 블로킹 없음

넌블로킹 방식에서 다음 코드는 클라이언트가 연결 요청을 하지 않으면 무한 루프를 돈다. 

```java
while(true) {
    SocketChannel socketChannel = serverSocketChannel.accept();
    ... 
}
```

accept() 메서드가 블로킹되지 않고 바로 리턴(null)되기 때문에, 클라이언트가 연결 요청을 보내기 전까지 while 블록 내 코드가 반복 실행된다.
CPU가 과도하게 소비될 수 있어, 넌블로킹 방식은 이벤트 리스너 역할을 하는 셀렉터(Selector)를 사용한다. 
Selector는 멀티 채널의 작업을 싱글 스레드에서 처리할 수 있도록 해주는 멀티플렉서(multiplexor) 역할을 한다. 

동작 방식
1. 채널은 Selector에 자신을 등록할 때 작업 유형을 키(SelectionKey)로 생성하고, Selector의 관심키셋(interest-set)에 저장시킨다.
2. 클라이언트가 처리 요청을 하면 Selector는 관심키셋에 등록된 키 중에서 작업 처리 준비가 된 키를 선택된 키셋(selected-set)에 별도로 저장한다. 
3. 그리고 작업 스레드가 선택된 키셋에 있는 키를 하나씩 꺼내어 키와 연관된 채널 작업을 처리한다. 
4. 작업 스레드가 선택된 키셋에 있는 모든 키를 처리하게 되면 선택된 키셋은 비워지고, Selector는 다시 관심키셋에서 작업 처리 준비가 된 키들을 선택해서 선택된 키셋을 채운다. 

![selector](selector.png)

작업 스레드는 꼭 하나일 필요는 없으며, 채널 작업 처리 시 스레드풀을 사용한다. 
작업 스레드가 블로킹되지 않기 때문에 적은 수의 스레드로 많은 양의 작업을 고속으로 처리할 수 있다. 


## 셀렉터(Selector) 사용 방법
### 셀렉터 생성과 등록
Selector는 정적 메서드인 open() 메서드를 호출해 생성한다. 

```java
try {
    Selector selector = Selector.open(); 
} catch (IOException e) {
    ...
}
```

Selector 등록 조건 
- SelectableChannel의 하위 채널 
    - ServerSocketChannel, SocketChannel, DatagramChannel 모두 등록 가능
- 넌블로킹으로 설정된 것만 가능

ServerSocketChannel을 넌블로킹으로 설정

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.configureBlocking(false); 
```

SocketChannel을 넌블로킹으로 설정

```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.configureBlocking(false); 
```

각 채널은 `register()` 메서드를 이용해 Selector에 등록한다. 
첫 번째 매개값은 Selector이고 두 번째 매개값은 채널의 작업 유형이다. 

```java
SelectionKey selectionKey = serverSocketChannel.register(Selector sel, int ops); 
SelectionKey selectionKey = socketChannel.register(Selector sel, int ops); 
```

아래는 두 번째 매개값으로 사용할 수 있는 작업 유형별 SelectionKey의 상수들이다. 

|SelectionKey의 상수|설명|
|-|-|
|OP_ACCEPT|ServerSocketChannel의 연결 수락 작업|
|OP_CONNECT|SocketChannel의 서버 연결 작업|
|OP_READ|SocketChannel의 데이터 읽기 작업|
|OP_WRITE|SocketChannel의 데이터 쓰기 작업|

register()는 채널과 작업 유형을 담고 있는 SelectionKey를 생성하고 Selector의 관심키셋에 저장한 후 해당 SelectionKey를 리턴한다. 

주의할 점은 동일한 SocketChannel로 두 가지 이상의 작업 유형을 등록할 수 없다.
(register()를 두 번 이상 호출할 수 없다.)
작업 유형이 변경되면 이미 생성된 SelectionKey를 수정해야 한다. 

SelectionKey는 작업 유형 변경, 첨부 객체 저장, 채널 등록 취소 등에 사용된다. 채널의 `keyFor()` 메서드를 통해 SelectionKey를 언제든지 얻을 수 있다. 

```java
SelectionKey key = socketChannel.keyFor(selector); 
```

### 선택된 키셋 
Selector는 `select()` 메서드를 통해 구동한다. 
- select()는 관심키셋에 저장된 SelectionKey로부터 작업 처리 준비가 되었다는 통보가 올 때까지 대기(블로킹)한다.
- 최소한 하나의 SelectionKey로부터 작업 처리 준비가 되었다는 통보가 오면 리턴한다. 
    - 리턴값은 통보를 해온 SelectionKey의 수이다. 

|리턴 타입|메서드명(매개 변수)|설명|
|-|-|
|int|select()|최소한 하나의 채널이 작업 처리 준비가 될 때까지 블로킹된다.|
|int|select(long timeout)|select()와 동일한데, 주어진 시간(ms) 동안만 블로킹한다.|
|int|selectNow()|호출 즉시 리턴된다. 작업 처리 준비된 채널이 있으면 채널 수를 리턴하고, 없다면 0을 리턴한다.|

select()가 리턴하는 경우는 다음과 같다. 
- 채널이 작업 처리 준비가 되었다는 통보를 할 때
- Selector의 wakeup() 메서드를 호출할 때
- select()를 호출한 스레드가 인터럽트될 때

ServerSocketChannel은 작업 유형이 하나밖에 없지만, SocketChannel은 상황에 따라 작업 유형을 변경할 수 있다. 
채널의 작업 유형이 변경되면 SelectionKey의 작업 유형을 `interestOps()` 메서드로 변경해야 한다.<br>
다음은 SelectionKey의 작업 유형을 OP_WRITE로 변경하는 코드이다. 

```java
selectionKey.interestOps(SelectionKey.OP_WRITE);
selector.wakeup();
```

SelectionKey의 작업 유형이 변경되면 Selector의 wakeup() 메서드를 호출해 블로킹되어 있는 select()를 즉시 리턴하고, 변경된 작업 유형을 감지하도록 select()를 다시 실행해야 한다. 

```java
int keyCount = selector.select();
if(keyCount > 0) {
    Set<SelectionKey> selectedKeys = selector.selectedKey();
}
```

위 코드에서는 select() 메서드가 1 이상의 값을 리턴할 경우, `selectedKey()` 메서드로 작업 처리 준비된 SelectionKey들을 얻는다. 
이 Set 컬렉션이 <u>선택된 키셋</u>이다. 

### 채널 작업 처리 
작업 스레드는 선택된 키셋에서 SelectionKey를 하나씩 꺼내어 작업 유형별로 채널 작업을 처리한다. 
다음 메서드를 통해 SelectionKey가 어떤 작업 유형인지 알아낼 수 있다. 

|리턴 타입|메서드명(매개 변수)|설명|
|boolean|isAcceptable()|작업 유형이 OP_ACCEPT인 경우 true 리턴|
|boolean|isConnectable()|작업 유형이 OP_CONNECT인 경우 true 리턴|
|boolean|isReadable()|작업 유형이 OP_READ인 경우 true 리턴|
|boolean|isWritable()|작업 유형이 OP_WRITE인 경우 true 리턴|

다음은 작업 스레드가 유형별로 채널 작업을 처리하는 방법이다. 

```java
Thread thread = new Thread() {
    @Override
    public void run() {
        while(true) {
            try {
                int keyCount = selector.select(); // 작업 처리 준비된 키 감지
                if(keyCount == 0) { continue; }
                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                
                Iterator<SelectionKey> iterator = selectedKeys.iterator(); 
                while(iterator.hasNext()) {
                    SelectionKey selectionKey = iterator.next(); // 키를 하나씩 꺼내옴
                    if(selectionKey.isAcceptable()) { // 연결 수락 작업 처리 }
                    else if(selectionKey.isReadable()) { // 읽기 작업 처리 }
                    else if(selectionKey.isWritable()) { // 쓰기 작업 처리 }
                    iterator.remove(); // 선택된 키셋에서 처리 완료된 SelectionKey 제거
                } 
            } catch (Exception e) {
                break;
            }
        }
    }
};
thread.start(); 
```

작업 스레드가 채널 작업을 처리하려면 채널 객체가 필요한데, SelectionKey의 `channel()` 메서드를 통해 얻을 수 있다. 

```java
ServerSocketChannel serverSocketChannel = (ServerSocketChannel) selectionKey.channel(); 
```

작업 스레드가 채널 작업을 처리하다 보면 채널 객체 외에 다른 객체가 필요할 수 있다. 
이러한 객체는 SelectionKey에 첨부해 두고 사용할 수 있다. 

```java
// 객체 첨부하기 
Client client = new Client(socketChannel); 
SelectionKey selectionKey = socketChannel.register(selector, SelectionKey.OP_READ);
selectionKey.attach(client); // 객체를 첨부

// 첨부된 객체 얻기 
if (selectionKey.isReadable()) {
    Client client = (Client)selectionKey.attachment(); //첨부된 객체 얻기 
}
```