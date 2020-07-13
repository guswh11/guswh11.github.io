---
layout: post
title: "제네릭(Generic), 제네릭 타입과 사용 방법"
categories: [java]
comments: true
tags:
  - Java
  - Generic
---
제네릭(Generic) 타입은 Java 5부터 새로 추가되었다.<br>
제네릭은 컬렉션, 람다식, 스트림, NIO에서 널리 사용되며, API 도큐먼트에도 제네릭 표현이 많기 떄문에 확실히 이해할 수 있어야 한다.<br>

제네릭은 클래스와 인터페이스, 메서드를 정의할 때 타입(type)을 파라미터(parameter)로 사용할 수 있도록 한다.

**제네릭 코드의 장점**
- 컴파일 시 강한 타입 체크 가능
    - 컴파일 시에 미리 타입을 강하게 체크해 에러를 사전에 방지
- 타입 변환(casting)을 제거
    - 비제네릭 코드는 불필요한 타입 변환을 해 프로그램 성능에 악영향을 미친다.
        ```java
        List list = new ArrayList();
        list.add("hello"); 
        String str = (String) list.get(0); //타입 변환
        ```
        
        위 코드를 다음과 같이 제네릭 코드로 수정하면 List에 저장되는 요소를 String으로 국한해 타입 변환이 불필요하다. 

        ```java
        List<String> list = new ArrayList<String>();
        list.add("hello"); 
        String str = list.get(0);
        ```

### 제네릭 타입(class<T>, interface<T>)
제네릭 타입은 타입을 파라미터로 가지는 클래스와 인터페이스를 말한다.<br>
제네릭 타입은 클래스나 인터페이스 이름 뒤에 "< >" 부호가 붙고, 사이에 타입 파라미터가 위치한다. 

```java
public class 클래스명<T> { ... }
public interface 인터페이스명<T> { ... }
```

타입 파라미터는 변수명과 동일한 규칙에 따라 작성할 수 있지만, 일반적으로 대문자 알파벳 한 글자(T)로 표현한다. 

#### 비제네릭 코드와의 비교 
다음 코드는 타입 파라미터를 사용해야 하는 이유를 보여준다. 

```java
public class Box {
    private Object object; 
    public void set(Object object) { this.object = object; }
    public Object get() { return object; }
}
```

Box 클래스의 필드 타입을 Object로 선언한 이유는, 필드에 모든 종류의 객체를 저장하고 싶어서이다.<br>
만약 필드에 저장된 원래 타입의 객체를 얻으려면 다음과 같이 강제 타입 변환을 해야 한다. 

```java
Box box = new Box(); 
box.set("hello"); // String 타입을 Object 타입으로 자동 타입 변환
String str = (String) box.get(); // Object 타입을 String 타입으로 강제 타입 변환
```

다음은 제네릭을 이용해 Box 클래스를 수정한 것이다. 

```java
public class Box<T> {
    private T t; 
    public T get() { return t; }
    public void set(T t) { this.t = t; }
}
```

T는 Box 클래스로 객체를 생성할 때 구체적인 타입으로 변경된다. 

```java
Box<String> box = new Box<String>(); 
box.set("hello"); 
String str = box.get(); 
```

이와 같이 제네릭은 클래스를 설계할 때 구체적인 타입을 명시하지 않고, 타입 파라미터로 대체했다가 실제 클래스가 사용될 때 구체적인 타입을 지정함으로써 타입 변환을 최소화시킨다. 

### 멀티 타입 파라미터(class<K,V,...>, interface<K,V,...>)
제네릭 타입은 두 개 이상의 멀티 타입 파라미터를 사용할 수 있는데, 이 경우 각 타입 파라미터를 콤마(,)로 구분한다. 

**제네릭 클래스**

```java
public class Product<T, M> {
    private T kind; 
    private M model; 

    public T getKind() { return this.kind; }
    public M getModel() { return this.model; }

    public void setKind(T kind) { this.kind = kind; }
    public void setModel(M model) { this.model = model; }
}
```

**제네릭 객체 생성**

```java
public class ProductExample {
    public static void main(String[] args) {
        Product<Tv, String> product = new Product<Tv, String>();
        product.setKind(new Tv());
        product.setModel("스마트Tv"); 
        Tv tv = product.getKind(); 
        String tvModel = product.getModel(); 
    }
}
```

자바 7부터는 다음과 같이 타입 파라미터를 유추해 자동으로 설정해준다. 

```java
Product<Tv, String> product = new Product<>(); 
```

### 제네릭 메서드(<T, R> R method(T t))
제네릭 메서드는 매개 타입과 리턴 타입으로 타입 파라미터를 갖는 메소드이다. 

제네릭 메서드를 선언하려면, 리턴 타입 앞에 < > 기호를 추가하고 타입 파라미터를 기술한 다음, 리턴 타입과 매개 타입으로 타입 파라미터를 사용하면 된다. 

```java
public <타입파라미터, ...> 리턴타입 메서드명(매개변수, ...) { ... }
```

제네릭 메서드는 두 가지 방식으로 호출할 수 있다. 타입 파라미터의 구체적인 타입을 명시적으로 지정해도 되고, 컴파일러가 매개값의 타입을 보고 구체적인 타입을 추정하도록 할 수 있다. 

```java
리턴타입 변수 = <구체적타입> 메서드명(매개값); // 명시적으로 구체적 타입을 지정
리턴타입 변수 = 메서드명(매개값); // 매개값을 보고 구체적 타입을 추정
```

**제네릭 메서드**

```java
public class Util {
    public static <T> Box<T> boxing(T t) {
        Box<T> box = new Box<T>(); 
        box.set(t); 
        return box
    }
}
```

제네릭 메서드 호출 

```java
public class BoxingMethodExample {
    public static void main(String[] args) {
        Box<Integer> box1 = Util.<Integer>boxing(100);
        int intValue = box1.get(); 

        Box<String> box2 = Util.boxing("홍길동");
        String strValue = box2.get(); 
    }
}
```

### 제한된 타입 파라미터(<T extends 최상위타입>)
타입 파라미터에 지정되는 구체적인 타입을 제한할 필요가 종종 있다. 예를 들어 숫자를 연산하는 제네릭 메서드는 매개값으로 Number 타입 또는 하위 클래스 타입(Byte, Short, Integer, Long, Double)의 인스턴스만 가져야 한다. 

제한된 타입 파라미터를 선언하려면 타입 파라미터 뒤에 extends 키워드를 붙이고 상위 타입을 명시하면 된다. 상위 타입은 클래스뿐만 아니라 인터페이스도 가능하다. 

```java
public <T extends 상위타입> 리턴타입 메서드(매개변수, ...) { ... }
```

타입 파라미터에 지정되는 구체적인 타입은 상위 타입이거나 상위 타입의 하위 또는 구현 클래스만 가능하다. 메서드의 중괄호 {} 안에서 타입 파라미터 변수로 사용 가능한 것은 상위 타입의 멤버(필드, 메서드)로 제한된다. 하위 타입에만 있는 필드와 메서드는 사용할 수 없다. 

### 와일드카드 타입(<?>, <? extends ...>, <? super ...>)
제네릭 타입을 매개값이나 리턴 타입으로 사용할 때 구체적인 타입 대신에 와일드카드를 다음과 같이 세 가지 형태로 사용할 수 있다. 
- 제네릭타입<?>: Unbounded Wildcards(제한 없음)
    - 타입 파라미터를 대치하는 구체적인 타입으로 모든 클래스나 인터페이스 타입이 올 수 있음
- 제네릭타입<? extends 상위타입>: Upper Bounded Wildcards(상위 클래스 제한)
    - 타입 파라미터를 대치하는 구체적인 타입으로 해당 타입과 하위 타입만 올 수 있음
- 제네릭타입<? super 하위타입>: Lower Bounded Wildcards(하위 클래스 제한)
    - 타입 파라미터를 대치하는 구체적인 타입으로 해당 타입과 상위 타입만 올 수 있다. 

// 사진넣기 

- Course<?>: 수강생은 모든 타입(Person, Worker, Student, HighStudent)이 될 수 있다. 
- Course<? extends Student>: 수강생은 Student와 HighStudent만 될 수 있다. 
- Course<? super Worker>: 수강생은 Worker와 Person만 될 수 있다. 

### 제네릭 타입의 상속과 구현 
제네릭 타입도 부모 클래스가 될 수 있다. 다음은 Product<T, M> 제네릭 타입을 상속해서 ChildProduct<T, M> 타입을 정의한다. 

```java
public class ChildProduct<T, M> extends Product<T, M> { ... }
```

자식 제네릭 타입은 추가적으로 타입 파라미터를 가질 수 있다. 다음은 세 가지 타입 파라미터를 가진 자식 제네릭 타입을 선언한 것이다. 

```java
public class ChildProductg<T, M, C> extends Product<T, M> { ... }
```

제네릭 인터페이스를 구현한 클래스도 제네릭 타입이 된다. 

```java
public interface Storage<T> {
    public void add(T item, int index); 
    public T get(int index); 
}
```

Storage<T> 타입을 구현한 StorageImpl 클래스도 제네릭 타입이어야 한다. 

```java
public class StorageImpl<T> implements Storage<T> {
    private T[] array; 

    public StorageImpl(int capacity) {
        this.array = (T[]) (new Object[capacity]); // 타입 파라미터로 배열을 생성하려면 new T[n] 형태로 생성할 수 없고 (T[])(new Object[n])으로 생성해야 한다. 
    }
}
```