# 토비의 봄 - Reactive Streams 02
토비의 봄 - Reactive Streams - Operators
## Operator
`Publisher`와 `Subscriber` 사이에서 데이터를 가공하는 역할?

`Publisher.subscribe(Subscriber)`
* `Publisher`가 `Subscriber`에게 데이터를 보낼꺼다.

```java
package com.naver.live02;  
  
import lombok.extern.slf4j.Slf4j;  
import org.reactivestreams.Publisher;  
import org.reactivestreams.Subscriber;  
import org.reactivestreams.Subscription;  
  
import java.util.List;  
import java.util.function.Function;  
import java.util.stream.Collectors;  
import java.util.stream.Stream;  
  
@Slf4j  
public class PubSub {  
    public static void main(String[] args) {  
        List<Integer> list = Stream  
                .iterate(1, a -> a + 1)  
                .limit(10)  
                .collect(Collectors.toList());  
  
        Publisher<Integer> pub = iterPub(list);  
        Publisher<String> reducePub = genericPub(pub, a -> "[" + a + "]");  
        reducePub.subscribe(logSub());  
    }  
  
    private static <T, R> Publisher<R> genericPub(Publisher<T> pub, Function<T, R> f) {  
        return new Publisher<R>() {  
            @Override  
            public void subscribe(Subscriber<? super R> sub) {  
                pub.subscribe(new DelegateSub<T, R>(sub) {  
                    @Override  
                    public void onNext(T t) {  
                        sub.onNext(f.apply(t));  
                    }  
  
                    @Override  
                    public void onComplete() {  
                        sub.onComplete();  
                    }  
                });  
            }  
        };  
    }  
  
    // reducePub도 Generic하게 바꿔보기  
  
    private static Publisher<Integer> iterPub(List<Integer> iter) {  
        return new Publisher<Integer>() {  
            @Override  
            public void subscribe(Subscriber<? super Integer> sub) {  
                sub.onSubscribe(new Subscription() {  
                    @Override  
                    public void request(long n) {  
                        try {  
                            iter.forEach(s -> sub.onNext(s));  
                            sub.onComplete();  
                        } catch (Throwable e) {  
                            sub.onError(e);  
                        }  
                    }  
  
                    @Override  
                    public void cancel() {  
  
                    }  
                });  
            }  
        };  
    }  
  
    private static <T> Subscriber<T> logSub() {  
        return new Subscriber<T>() {  
            @Override  
            public void onSubscribe(Subscription s) {  
                log.info("onSubscribe");  
                s.request(Long.MAX_VALUE);  
            }  
  
            @Override  
            public void onNext(T t) {  
                log.info("onNext={}", t);  
            }  
  
            @Override  
            public void onError(Throwable t) {  
                log.info("onError={}", t);  
            }  
  
            @Override  
            public void onComplete() {  
                log.info("onComplete");  
            }  
        };  
    }  
}
```

```java
@Slf4j  
@RestController  
public static class HelloController {  
    @RequestMapping("/hello")  
    public Publisher<Integer> hello() {  
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

`subscribe`에 대한 부분은 스프링이 알아서 처리해준다.
다만, 해당 코드로 호출하면 `[1,1]` 이렇게 1이 2번 호출되게 되는데, 이는 `request(Long.MAX_VALUE)`로 `unbound request`가 오는 것이 아니라 netty 내부에서 `request(1)` 한번 `request(127)` 한번 이렇게 2번 호출하기 때문이다.

따라서 한번만 호출되게 할꺼면 다른 `mono`라는 것을 사용하거나 해당 코드를 굳이 써야한다면 flag를 사용하거나 count하는 방식으로 사용하면 괜찮을 것 같다.

1000개짜리 `list`를 만들어서 요청해도 128개만 오는데 그럼 다 받으려면 어떻게 해야되지?

```java
// TODO : 얘는 다 온다 이유가 뭐지?
@Slf4j  
@RestController  
public static class TestController {  
    @RequestMapping("/test")  
    public Publisher<Integer> test(Integer bound) {  
        Queue<Integer> list = new ArrayDeque<>();  
        IntStream.range(1, bound).forEach(list::add);  
  
        return Flux.create(sub -> {  
            list.forEach(sub::next);  
            sub.complete();  
        });  
    }  
}
```