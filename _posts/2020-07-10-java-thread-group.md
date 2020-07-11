---
layout: post
title: "스레드 그룹(ThreadGroup) 개념 및 사용법"
categories: [java]
comments: true
tags:
  - Java
  - Thread
  - ThreadGroup
---
## 스레드 그룹 
스레드 그룹(ThreadGroup): 관련된 스레드를 묶어서 관리할 목적으로 이용됨

- system 스레드 그룹
    - JVM이 실행될 때 만들어짐
    - JVM 운영에 필요한 스레드들이 생성되면 system 스레드 그룹에 포함됨
- main 스레드 그룹
    - system의 하위 스레드 그룹
    - 메인 스레드가 포함됨 

스레드는 반드시 하나의 스레드 그룹에 포함되는데, 명시적으로 스레드 그룹에 포함시키지 않으면 기본적으로 부모 스레드와 같은 스레드 그룹에 속하게 된다.<br>대부분의 작업 스레드는 메인 스레드가 생성하므로 main 스레드 그룹에 속하게 된다. 

### 스레드 그룹 이름 얻기
현재 스레드가 속한 스레드 그룹의 이름은 다음과 같이 얻을 수 있다. 

```java
ThreadGroup group = Thread.currentThread().getThreadGroup();
String groupName = group.getName(); 
```

`getAllStackTraces()`를 이용하면 프로세스 내에서 실행하는 모든 스레드에 대한 정보를 얻을 수 있다. 

```java
public class ThreadInfoExample {
    public static void main(String[] args) {
        AutoSaveThread autoSaveThread = new AutoSaveThread();
        autoSaveThread.setName("AutoSaveThread");
        autoSaveThread.setDaemon(true);
        autoSaveThread.start();

        Map<Thread, StackTraceElement[]> map = Thread.getAllStackTraces(); 
        Set<Thread> threads = map.keySet();

        for(Thread thread : threads){
            System.out.println("Name : " + thread.getName() + ((thread.isDaemon()) ? "[Daemon]" : "[Main]"));
            System.out.println("\t" + "Group : " + thread.getThreadGroup().getName());
            System.out.println();
        }
    }
}
```

### 스레드 그룹 생성 
스레드 그룹은 다음과 같이 명시적으로 만들 수 있다. 

```java
ThreadGroup tg = new ThreadGroup(String name); 
ThreadGroup tg = new ThreadGroup(Threadgroup parent, String name); 
```

생성 시 부모 스레드 그룹을 지정하지 않으면, 현재 스레드가 속한 그룹의 하위 그룹으로 생성된다.

Thread 객체를 생성할 때 생성자 매개값으로 스레드 그룹을 지정하면, 해당 스레드 그룹에 스레드를 포함시킬 수 있다. 

```java 
Thread t = new Thread(ThreadGroup group, Runnable target);
Thread t = new Thread(ThreadGroup group, Runnable target, String name);
Thread t = new Thread(ThreadGroup group, Runnable target, String name, long stackSize);
Thread t = new Thread(ThreadGroup group, String name);
```

### 스레드 그룹의 일괄 interrupt()
스레드 그룹에서 제공하는 interrupt() 메서드를 통해 그룹 내 포함된 모든 스레드들을 일괄 interrupt 할 수 있다. 

![thread group interrupt](thread-group-interrupt.png)
