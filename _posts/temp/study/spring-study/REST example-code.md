# REST API 만들기 
## REST 컨트롤러 사용 
클라이언트가 웹인 경우 JSON 데이터를 처리하는 것이 수월하므로,<br>최근 대부분의 REST API는 JSON 포맷으로 응답하도록 개발된다. 
- 특정 경로로 요청을 보냈을 때 응답을 JSON 포맷으로 제공하는 API

    ```java
    public class Todo {
        private long id;
        private String title;

        public Todo(long id, String title){
            this.id = id;
            this. title = title;
        }

        public long getId() {
            return id;
        }

        public void setId(long id) {
            this.id = id;
        }

        public String getTitle() {
            return title;
        }

        public void setTitle(String title) {
            this.title = title;
        }
    }

    ```
    
    아래는 URL을 요청하면 Todo 클래스의 인스턴스를 생성해서 JSON으로 보여주는 컨트롤러이다. 

    ```java
    @RestController
    @RequestMapping(value="/basic")
    public class BasicController {
        private final AtomicLong counter = new AtomicLong();

        @RequestMapping(value="/todo")
        public Todo basic(){
            return new Todo(counter.incrementAndGet(), "라면사오기");
        }
    }
    ```
    ![json-answer](json-answer.png)
    
## HTTP Method 사용 

## URI 예제

## HTTP header 활용
