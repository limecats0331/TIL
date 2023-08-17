```java 
@Slf4j  
@RestController  
public static class HelloController {  
    @RequestMapping("/hello")  
    public Publisher<Integer> hello(Integer bound) {  
        return new Publisher<Integer>() {  
            @Override  
            public void subscribe(Subscriber<? super Integer> s) {  
                s.onSubscribe(new Subscription() {  
                    @Override  
                    public void request(long n) {  
                        s.onNext(1);  
                        s.onComplete();  
                    }  
  
                    @Override  
                    public void cancel() {  
  
                    }  
                });  
            }  
        };  
    }  
}
```

해당 코드를 사용해서 `localhost:8080/hello`를 호출해보면 `[1,1]` 이렇게 데이터가 2번 발송된다.
그래서 뭐가 문제인지 찾아보니깐 스프링에서 request를 `request(1)` 한번, `request(127)` 한번 해서 총 2번 보내기 떄문에 그런 것 같다.
`unbound`로 보내지 않고 쪼개서 보내니깐 호출이 2번 되는 것