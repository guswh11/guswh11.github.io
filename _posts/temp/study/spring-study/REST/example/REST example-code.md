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
    클라이언트가 /basic/todo 경로로 요청을 보내면, 다음과 같은 JSON 형태의 응답을 확인할 수 있다. 
    ![json-answer](json-answer.png)
    `RestController` 어노테이션은 스프링 4.1부터 추가된 어노테이션으로, 기존 컨트롤러 어노테이션에서 `ResponseBody` 어노테이션을 추가한 것이다.<br>그래서 RestController 어노테이션을 사용하면 별도로 ResponseBody 어노테이션을 사용하지 않고 REST API를 구현할 수 있다.<br>ResponseBody 어노테이션은 리턴값을 view를 통해서가 아닌, HTTP Response Body에 직접 입력되게 한다. 이때 리턴값의 데이터 타입에 따라 MessageConverter에서 변환이 이뤄진 후 쓰여지게 되는데, 그 중 MappingJacksonHttpMessageConverter을 통해 JSON으로 결과가 변환 작업이 이루어진다. 
    ` RestController 어노테이션 사용을 통해 JSON으로 리턴값 자동 변환이 가능하다. `

    ---

    RestController를 사용하지 않는다면, 별도로 ResonsceBody 어노테이션 사용이 필요하다. 
    아래 컨트롤러 또한 위 스크린샷과 같은 결과를 보인다.
    ```java
    @Controller
    @RequestMapping(value="/basic")
    public class BasicController {
        private final AtomicLong counter = new AtomicLong();

        @ResponseBody
        @RequestMapping(value="/todo")
        public Todo basic(){
            return new Todo(counter.incrementAndGet(), "라면사오기");
        }
    }
    ```

    ResponseBody 어노테이션을 사용하지 않을 경우, 리턴값은 view를 통해 보여져야 하므로 whitelabel error가 뜨게 된다. 

## HTTP Method 사용 

HTTP Method란, HTTP 프로토콜 사용 시의 호출 방식을 말한다. <br>GET은 조회, POST는 생성, PUT은 업데이트 요청, DELETE는 제거를 뜻한다. 

methods | GET     | POST | PUT | DELETE
------- | ------- | ---- | --- | ------
/users  | 전체 사용자 조회 | 사용자 추가 | 사용자 추가 / 수정 | 사용자 전체 삭제 
/users/hyunjin| 사용자 'hyunjin' 조회 | 405 ERROR | 사용자 'hyunjin' 수정 | 사용자 'hyunjin' 삭제

GET은 같은 요청에 대해서 같은 응답임을 보장한다. 
POST의 경우, URL을 통한 직접 호출은 불가능하다.<br>POST 메서드를 사용하는 폼을 만들거나 별도의 도구를 사용해야 한다.
<br>GET, POST만 사용하고도 대부분의 기능 구현이 가능하다. 

- HTTP Method를 활용한 게시판 controller 예제 코드
    ```java
    @RestController
    public class PostController {
        @Autowired
        PostRepository postRepository;

        @GetMapping("/post")
        public ModelAndView getAllPost() throws Exception{
            List<Post> postList = postRepository.findAll();
            return new ModelAndView("board/home","postList",postList);
        }

        @PostMapping("/post/{id}")
        public Post updatePost(@PathVariable String id, @RequestBody Post newPost){
            Long postId = Long.parseLong(id);
            System.out.println(newPost);
            System.out.println(postId);

            Optional<Post> post = postRepository.findById(postId);

            if(post.isPresent()){
                post.get().setTitle(newPost.getTitle());
                post.get().setContent(newPost.getContent());
                post.get().setDate();
            }

            postRepository.save(post.get());
            return post.get();
        }

        @PutMapping("/post")
        public Post addPost(@RequestBody Post post){
            System.out.println(post.getTitle());
            post.setDate();
            Post newPost = postRepository.save(post);
            return newPost;
        }

        @DeleteMapping("/post/{id}")
        public String deletePost(@PathVariable String id){
            Long postId = Long.parseLong(id);
            postRepository.deleteById(postId);

            return "Delete success";
        }
    }
    ```
    
## HTTP 상태 코드 사용 
**성공 응답 코드**
- 200 : [OK]
- 201 : [Created]
    - 200과 달리 요청에 성공하고 새로운 리소스를 만든 경우에 응답한다.
    - POST, PUT에 사용한다.
- 202 : [Accepted]
    - 클라이언트 요청을 받은 후, 요청은 유효하나 서버가 아직 처리하지 않은 경우에 응답한다. (비동기 작업)
        - 요청에 대한 응답이 일정 시간 후 완료되는 작업의 경우<br>작업 완료 후 클라이언트에 알릴 수 있는 server push 작업을 하거나, 클라이언트가 해당 작업의 진행 상황을 조회할 수 있는 URL을 응답해야 한다.

    ```JSON
    HTTP/1.1 202 Accepted
    {
        "links": [
            {
                "rel": "self",
                "method": "GET",
                "href":  "https://api.test.com/v1/users/3"
            }
        ] 
    }
    ```

- 204 : [No Content]
    - 응답 body가 필요 없는 자원 삭제 요청(DELETE) 같은 경우 응답한다.
    - 200 응답 후 body에 null, {}, [], false로 응답하는 것과 다르다.<br>(204의 경우 HTTP body가 아예 없음)

**리다이렉션 응답 코드**
- 301 : [Moved Permanently]
    - 클라이언트가 요청한 리소스에 대한 URI가 변경되었을 때 사용
    - 응답 시 Location header에 변경된 URI를 적어줘야 함
- 303 : [See Other] 
    - 요청한 자원이 임시 주소에 존재
- 304 : [Not Modified]
    - 요청한 자원이 변경되지 않았으므로 클라이언트에서 캐싱된 자원을 사용하도록 권고
    - ETag와 같은 정보를 활용하여 변경 여부를 확인

**실패 응답 코드**
- 400 : [Bad Request]
    - 클라이언트 요청이 미리 정의된 파라미터 요구사항을 위반한 경우
    - 파라미터의 위치(path, query, body), 사용자 입력 값, 에러 이유 등을 반드시 알린다

    case 1
    ```
    {
        "message" : "'name'(body) must be Number, input 'name': test123"
    }
    ```
    case 2
    ```
    {
        "errors": [
            {
                "location": "body",
                "param": "name",
                "value": "test123",
                "msg": "must be Number"
            }
        ]
    }
    ```
- 401 : [Unauthorized]
- 403 : [Forbidden]
    - 해당 요청은 유효하나 서버 작업 중 접근이 허용되지 않은 자원을 조회하려는 경우
    - 접근 권한이 전체가 아닌 일부만 허용되어 요청자의 접근이 불가한 자원에 접근 시도한 경우 응답한다.
- 404 : [Not Found]
- 405 : [Method Not Allowed]
    - 405 code는 404 code와 혼동될 수 있기 때문에 룰을 잘 정하고 시작한다.
    - POST /users/1의 경우 404로 응답한다고 생각할 수 있지만, 경우에 따라 405로 응답할 수 있다. <br>/users/:id URL은 GET, PATCH, DELETE method는 허용되고 POST는 불가한 URL이다.
    - 만약 id가 1인 사용자가 없는 경우엔 404로 응답하지만(GET, PATCH, DELETE의 경우), POST /users/1는 /users/:id URL이 POST method를 제공하지 않기 때문에 405로 응답하는 게 옳다.
- 409 : [Conflict]
    - 해당 요청의 처리가 비지니스 로직상 불가능하거나 모순이 생긴 경우
    e.g.) DELETE /users/hak의 경우, 비지니스 로직상 사용자의 모든 자원이 비어있을 때만 사용자를 삭제할 수 있는 규칙이 있을 때 409로 응답한다.
    ```
    409 Conflict
    {   
        "message" : "first, delete connected resources."
        "links": [
            {
                "rel": "posts.delete",
                "method": "DELETE",
                "href":  "https://api.test.com/v1/users/hak/posts"
            },
            {
                "rel": "comments.delete",
                "method": "DELETE",
                "href":  "https://api.test.com/v1/users/hak/comments"
            }
        ]
    }
    ```
- 429 : [Too Many Requests]
    - DoS, Brute-force attack 같은 비정상적인 접근을 막기 위해 요청의 수를 제한한다.

**서비스 장애 코드**
- 500
    - API Server level에선 나지 않음
    - API Server를 서빙하는 웹서버(apache, nginx)가 오류일 때 가능
- 301
    - 클라이언트가 요청한 리소스에 대한 URI가 변경되었을 때 사용
    - 응답 시 Location header에 변경된 URI를 적어줘야 함

## URI 예제

## HTTP header 활용


## Reference 
RESTful API 설계 가이드[이상학의 개발블로그]: <https://sanghaklee.tistory.com/57>

