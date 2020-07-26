---
layout: post
title: "Java-스트림(Stream)"
categories: [java]
comments: true
tags:
  - Java
  - Stream
---

## 스트림(Stream)
스트림은 컬렌션의 저장 요소를 하나씩 참조해서 람다식으로 처리할 수 있도록 해주는 반복자이다. 

### 반복자 스트림

```java
List<String> list = Arrays.asList("a", "b", "c");
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()) {
    String alphabet = iterator.next();
    System.out.println(alphabet);
}
```

위 코드를 스트림을 사용해서 변경하면 다음과 같다. 

```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream(); 
stream.forEach(alphabet->System.out.println(alphabet));
```

forEach() 메서드는 다음과 같이 Consumer 함수적 인터페이스 타입의 매개값을 가지므로 컬렉션의 요소를 소비할 코드를 람다식으로 기술할 수 있다.

```java
void forEach(Consumer<T> action)
```

### 스트림의 특징
스트림은 Iterator와 비슷한 역할을 하는 반복자이지만, 많은 차이를 가지고 있다. 

**람다식으로 요소 처리 코드를 제공한다.**<br>
스트림이 제공하는 대부분의 요소 처리 메서드는 함수적 인터페이스 매개 타입을 가지기 때문에 람다식 또는 메서드 참조를 이용해 매개값을 전달할 수 있다.

**내부 반복자를 사용하므로 병렬 처리가 쉽다.**
    
![external/internal iterator](external-internal-iterator.png)
    
- 외부 반복자(external iterator): 개발자가 코드로 직접 컬렉션의 요소를 반복해서 가져오는 코드 패턴<br>
ex) index 이용 for문, Iterator 이용 while문

- 내부 반복자(internal iterator): 컬렉션 내부에서 요소들을 반복시키고, 개발자는 요소당 처리해야 할 코드만 제공하는 코드 패턴 
    - 개발자는 요소 처리 코드에만 집중 가능
    - 내부 반복자가 요소들의 반복 순서를 변경하거나, 멀티 코어 CPU 활용을 위해 요소들을 분배시켜 병렬 작업을 도와줌

**병렬 처리**: 나눠진 서브 작업들을 분리된 스레드에서 병렬적으로 처리하는 것

**병렬 처리 스트림**을 이용하면 런타임 시 하나의 작업을 서브 작업으로 자동으로 나누고, 서브 작업의 결과를 자동으로 결합해서 최종 결과물을 생성한다. 

![internal stream](internal-iterator.png)

```java 
import java.util.Arrays;
import java.util.List; 
import java.util.stream.Stream;

public class ParallelExample {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("a", "b", "c", "d");

        // 순차 처리 
        Stream<String> stream = list.stream();
        stream.forEach(ParallelExample :: print);
        System.out.println();

        // 병렬 처리
        Stream<String> parallelStream = list.parallelStream();
        parallelStream.forEach(ParallelExample :: print);
    }

    public static void print(String str) {
        System.out.println(str+" : "+Thread.currentThread().getName());
    }
}
```

실행 결과 

```
a : main
b : main
c : main
d : main

c : main
d : main
b : ForkJoinPool.commonPool-worker-3
a : ForkJoinPool.commonPool-worker-3
```

**중간 처리와 최종 처리를 할 수 있다.**<br>
중간 처리 - 매핑, 필터링, 정렬 수행 
최종 처리 - 반복, 카운팅, 평균, 총합 등의 집계 처리 수행

![stream pipeline](stream-pipeline.png)

다음 예제는 List에 저장되어 있는 Student 객체를 중간 처리에서 score 필드값으로 매핑하고, 최종 처리에서 score의 평균값을 산출한다. 

```java
public class MapAndReduceExample {
    public static void main(String[] args) {
        List<Student> studentList = Arrays.asList(
            new Student("학생1", 10),
            new Student("학생2", 20),
            new Student("학생3", 30)
        );

        double avg = studentList.stream()
        // 중간처리(학생 객체를 점수로 매핑)
        .mapToInt(Student :: getScore)
        // 최종처리(평균 점수)
        .average()
        .getAsDouble();

        System.out.println("평균 점수 : "+avg); 
    }
}
```