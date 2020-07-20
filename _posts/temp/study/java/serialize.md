# 직렬화(Serialize)
**직렬화(Serialize)** 
- 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 변환하는 기술
- JVM(Java Virtual Machine 이하 JVM)의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 바이트 형태로 변환하는 기술
- 다차원의 자료를 파일로 저장하거나 네트워크로 보내기에 알맞게 일차원으로 펼치고 다시 원래대로 되돌리는 것

**역직렬화(Deserialize)**
- 바이트 형태로 변환된 데이터를 원래대로 객체나 데이터로 변환하는 기술
- 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태

![serialize/deserialize](serialize-deserialize.jpg)

### 직렬화 조건
자바는 `Serializable` 인터페이스를 구현한 클래스만 직렬화할 수 있도록 제한하고 있다. 

```java
public class XXX implements Serializable { }
```

- 객체를 직렬화하면 필드들은 바이트로 변환된다.
    - 역직렬화할 때에는 필드의 값만 복원된다. 
- 생성자 및 메서드는 직렬화에 포함되지 않는다.
- `static` 또는 `transient`가 붙어 있을 경우에는 직렬화가 되지 않는다. 

```java
public class ClassA implements Serializable {
    // 직렬화에 포함
    int field1;
    Class field2 = new ClassB();

    // 직렬화에서 제외
    static int field3; 
    transient int field4; 
    
    public ClassA(int field1) {
        this.field1 = field1;
    }
} 
```

### 직렬화 방법 
자바 직렬화는 `java.io.ObjectOutputStream` 객체를 이용한다. 

```java
ClassA classA = new ClassA(1); 
byte[] serializedClass;
try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
        try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
            oos.writeObject(classA);
            serializedClass = baos.toByteArray();
        }
    }
    // 바이트 배열로 생성된 직렬화 데이터를 base64로 변환
    System.out.println(Base64.getEncoder().encodeToString(serializedClass));
}
```

아래는 직렬화해서 파일로 저장하는 코드이다. 

```java
ClassA classA = new ClassA(1); 
try (FileOutputStream fos = new FileOutputStream("C:/Temp/Object.dat")) {
    try (ObjectOutputStream oos = new ObjectOutputStream(fos)) {
        oos.writeObject(classA);
        oos.flush(); 
        oos.close();
        fos.close();
    }
}
```

### 역직렬화 조건
- 직렬화 대상이 된 객체의 클래스가 클래스 패스에 존재해야 하며 import 되어 있어야 한다.
- 직렬화했을 때와 같은 클래스를 사용해야 한다. 
    - 클래스의 이름이 같더라도 내용이 변경되면, 역직렬화는 다음과 같은 예외가 발생한다. 

    ```
    java.io.InvalidClassException: XXX; local class incompatible: stream classdesc
    serialVersionUID = -9130799490637378756, local class serialVersionUID = ~
    ```

    - 직렬화할 때와 역직렬화할 때 사용된 클래스의 serialVersionUID가 같아야 한다. 

만약 불가피하게 클래스의 수정이 필요하다면 클래스 작성 시 다음과 같이 serialVersionUID를 명시적으로 선언하면 된다. 

```java
public class XXX implements Serializable {
    static final long serialVersionUID = "정수값";
    ...
}
```

### 역직렬화 방법 
```java 
// 직렬화 예제에서 생성된 base64 데이터 
String base64Class = "...생략";
byte[] serializedClass = Base64.getDecoder().decode(base64Class);
try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedClass)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
        Object objectClass = ois.readObject();
        ClassA classA = (ClassA) objectClass;
    }
}
```

```java
try (FileInputStream fis = new FileInputStream("C:/Temp/Object.dat")) {
    try(ObjectInputStream ois = new ObjectInputStream(fis)) {
        ClassA classA = (ClassA) ois.readObject();
    }
}
```

## 직렬화 사용 이유 
아래 링크 참고 
<https://woowabros.github.io/experience/2017/10/17/java-serialize.html>


#### Reference 
<https://woowabros.github.io/experience/2017/10/17/java-serialize.html><br>
<https://j.mearie.org/post/122845365013/serialization#_=_><br>
