# 람다식(Lambda Expressions)
자바는 함수적 프로그래밍을 위해 자바 8부터 람다식을 지원한다.<br>
람다식은 익명 함수(anonymous function)를 생성하기 위한 식으로, 매개 변수를 가진 코드 불록 형태이지만 런타임 시에는 익명 구현 객체를 생성한다.

```java
// 익명 구현 객체
Runnable runnable = new Runnable() {
    public void run() { ... }
};

// 람다식
Runnable runnable = () -> { ... };
```

자바 8 이전
- '메서드'라는 함수 형태가 존재하지만, 객체를 통해서만 접근 가능
- 메서드 그 자체를 변수로 사용하지는 못함 

람다식 도입 이후
- 함수를 변수처럼 사용
    - 다른 메서드의 인자로 전달 가능
    - 메서드가 리턴값이 될 수 있음

장점
- 코드가 간결해짐
- 

#### Reference 
<https://m.blog.naver.com/2feelus/220695347170><br>