# 토비의 봄 - Reactive Streams 03
토비의 봄 - Reactive Streams - Schedulers

지금까지의 코드들은 하나의 `main thread`에서 모든 작업이 처리된다. 하지만 이는 매우 비효율적이다.

`Scheduler`를 지정하는 방법이 크게 2가지 있는데 하나는 `subscribeOn`과 다른 하나는 `publishOn`이다.

`subscribeOn` 
* `Operator`이고 지정한 스레드에서 작업이 이뤄지도록 한다.
* `Publisher`가 아주 느린 경우 사용

```java
Publisher<Integer> subscribeOnPub = sub -> {  
    ExecutorService es = Executors.newSingleThreadExecutor();  
    es.execute(() -> pub.subscribe(sub));  
};
```

`publishOn`
* 데이터를 보내는 역할을 `main thread`에서 이루어진다.
* `subscriber`가 작업하는 부분들 별도의 스레드로 관리
* `subscriber`가 `consume`하는 작업이 느린 경우에 사용

```java
Publisher<Integer> publishOnPub = sub -> {  
	pub.subscribe(new Subscriber<Integer>() {  
		//표준에서도 하나의 publisher가 던진 데이터는 하나의 스레드가 받게 되어있다.
		ExecutorService es = Executors.newSingleThreadExecutor();  

		@Override  
		public void onSubscribe(Subscription s) {  
			sub.onSubscribe(s);  
		}  

		@Override  
		public void onNext(Integer integer) {  
			es.execute(() -> sub.onNext(integer));  
		}  

		@Override  
		public void onError(Throwable t) {  
			es.execute(() -> sub.onError(t));  
		}  

		@Override  
		public void onComplete() {  
			es.execute(() -> sub.onComplete()); 
		}  
	});  
};
```

두 개를 다 걸 수도 있다.

`user thread`와 `daemon thread` 두 종류의 스레드가 존재하는데 보통 스레드는 `user thread` 하지만, `Flux.interval` 같은 애들은 `timer`라는 `daemon thread`를 사용하는데 이때, `JVM`이 `User thread`없이 `Daemon Thread`만 존재하면 프로그램을 종료시킨다.

```java
// 바로 종료됨
Flux.interval(Duration.ofMillis(500))  
		.log()  
		.subscribe(System.out::println);

// 바로 종료되지 않음
Flux.interval(Duration.ofMillis(500))  
		// 혹은 sleep을 건다.
        .subscribeOn(Schedulers.newSingle("time"))  
        .log()  
        .subscribe(System.out::println);
```

## Interval과 take 구현
```java
package com.naver.live03;  
  
import lombok.extern.slf4j.Slf4j;  
import org.reactivestreams.Publisher;  
import org.reactivestreams.Subscriber;  
import org.reactivestreams.Subscription;  
  
import java.util.concurrent.Executors;  
import java.util.concurrent.ScheduledExecutorService;  
import java.util.concurrent.TimeUnit;  
  
@Slf4j  
public class IntervalEx {  
    public static void main(String[] args) {  
        Publisher<Integer> pub = sub -> {  
            sub.onSubscribe(new Subscription() {  
                int no = 0;  
                boolean isCanceled = false;  
  
                @Override  
                public void request(long n) {  
                    ScheduledExecutorService exec = Executors.newSingleThreadScheduledExecutor();  
                    exec.scheduleAtFixedRate(() -> {  
                        if (isCanceled) {  
                            exec.shutdown();  
                            return;                        }  
                        sub.onNext(no++);  
                    }, 0, 500, TimeUnit.MILLISECONDS);  
                }  
  
                @Override  
                public void cancel() {  
                    isCanceled = true;  
                }  
            });  
        };  
  
        Publisher<Integer> takePub = sub -> {  
            pub.subscribe(new Subscriber<Integer>() {  
                final int MAX_COUNT = 10;  
                Subscription subscription;  
                int count = 0;  
  
                @Override  
                public void onSubscribe(Subscription s) {  
                    subscription = s;  
                    sub.onSubscribe(s);  
                }  
  
                @Override  
                public void onNext(Integer integer) {  
                    if (++count >= MAX_COUNT) {  
                        subscription.cancel();  
                    }  
                    sub.onNext(integer);  
                }  
  
                @Override  
                public void onError(Throwable t) {  
                    sub.onError(t);  
                }  
  
                @Override  
                public void onComplete() {  
                    sub.onComplete();  
                }  
            });  
        };  
  
        takePub.subscribe(new Subscriber<Integer>() {  
            @Override  
            public void onSubscribe(Subscription s) {  
                log.info("SchedulerEx.onSubscribe");  
                s.request(Long.MAX_VALUE);  
            }  
  
            @Override  
            public void onNext(Integer integer) {  
                log.info("SchedulerEx.onNext = {}", integer);  
            }  
  
            @Override  
            public void onError(Throwable t) {  
                log.info("SchedulerEx.onError");  
            }  
  
            @Override  
            public void onComplete() {  
                log.info("SchedulerEx.onComplete");  
            }  
        });  
    }  
}
```

