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

