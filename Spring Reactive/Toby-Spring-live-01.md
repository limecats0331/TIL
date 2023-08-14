# 토비의 봄 - Reactive Streams 01

`Reactive Programming`?
외부에서 어떠한 이벤트가 발생하면 그게 대응하는 방식으로 동작하는 것

`Reactive Streams` - 표준(`JVM` 언어 쪽)

## Duality

`java`의 `for-each`는 `collection`을 넣는 것이 아니라 `iterator`를 구현한 것을 넣는다.

```java
//use iterable
Iterable<Integer> list = Arrays.asList(1, 2, 3, 4, 5);  
  
for (Integer i : list) { // for-each  
    System.out.println(i);  
}
```

```java
//iterator를 직접 구현
//iterable은 functional interface라서 람다로 치환
Iterable<Integer> iter = () ->  
		//람다로 변환한 함수는 Iterator하나만 리턴하니깐 생략
		//굳이 클래스까지 만들 필요는 없으니깐 그냥 익명 클래스로
        new Iterator<>() {  
            int i = 0;  
            final static int MAX = 10;  
  
            public boolean hasNext() {  
                return i < MAX;  
            }  
  
            public Integer next() {  
                return ++i;  
            }  
        };  
  
for (Integer i : iter) {  
    System.out.println(i);  
}

//java 1.5 이하에서는 이렇게 사용했다.
//혹은 while문으로
for (Iterator<Integer> it = iter.iterator(); it.hasNext(); ) {  
    System.out.println(it.next());  
}
```


`Iterable` <--> `Observable` (`duality`)
기능은 같은데 방향이 반대?

* `Iterable` : `Pull` 개념,  사용하는 쪽에서 가져와서 사용
* `Observale` : `Push` 개념, 사용할 쪽으로 가져다 주는 방식

`Observer`를 `Observable`에 등록하면 `Observable`가 `Observer`에게 이벤트가 발생하면 이벤트나 데이터를 보낸다. (`notify`한다.) 여기서 `Oberser`는 여러 개가 될 수 있다.

	`Observable`을 `Publisher`라고도 하고 `Observer`를 `subscriber`라고도 한다.

위의 코드를 `Observable`하고 구현하면 다음과 같다.

```java
@SuppressWarnings("deprecation")  
public class Ob {  
  
    public static void main(String[] args) {  
        Observer ob = new Observer() {  
            @Override  
            public void update(Observable o, Object arg) {  
                System.out.println(arg);  
            }  
        };  
  
        IntObservable io = new IntObservable();  
        io.addObserver(ob);  
  
        io.run();  
    }  
  
    static class IntObservable extends Observable implements Runnable {  
        @Override  
        public void run() {  
            for (int i = 1; i <= 10; i++) {  
                setChanged();  
                notifyObservers(i); //push  
            }  
        }  
    }  
}
```

`Observable` 사용하면 여러 `Oberser`에 한번에 `boardcast`도 할 수 있다.
또한, `main thread`를 점유하지 않고 별도의 `thread`가 `Observable`를 점유해서 사용할 수 있도록 할 수 있다.

하지만 `Observer Pattern`도 2가지가 존재하지 않는다.
1. 데이터를 다 보냈다는 `Complete`의 개념
2. 에러 / 예외에 대한 처리 방식

이것까지 확장해서 만든 `Obervable`이 `Reactive Programming`의 한 축을 담당한다.

## Observer Pattern

`Reactive Streams`라는 곳에서 제공하는 표준 `API`가 있다.
`ReactvieX` 같은 곳도 여기에 포함되어있다.

* `Publisher`
	* **한계가 없고 연속된 순서를 가지는 데이터**를 **제공**한다.
	* `Subscriber`가 받을 수 있도록 제공해야 한다.
	* `Publisher.subscribe(Subscriber)`를 통해서 구독한다.
		* `Publisher`는 `Subscriber`에게 다음과 같은 시그널을 반드시 보내야한다.
	* `onSubscribe onNext* (onError | onComplete)?`
		* `onSubsribe` : 필수
		* `onNext` : 0회 이상
		* `onError`, `onComplete` : 둘 중에 하나만 베타적으로 옵셔널하게
* `Subscriber`
* `Subscription`
	* `Publisher`와 `Subscriber` 사이를 중계해주는 구독이라는 정보
	* 그냥 정보를 다 받을지, 혹은 n개만 받고 나중에 받을지 등을 가지고 있음
* `Processor`

```java
//Pubsub 기본 동작

package com.naver.live;  
  
import java.util.Arrays;  
import java.util.Iterator;  
import java.util.concurrent.Flow;  
import java.util.concurrent.Flow.Publisher;  
import java.util.concurrent.Flow.Subscriber;  
import java.util.concurrent.Flow.Subscription;  
  
public class Pubsub {  
    public static void main(String[] args) {  
        Iterable<Integer> iter = Arrays.asList(1, 2, 3, 4, 5);  
  
        Publisher pub = new Publisher() {  
            @Override  
            public void subscribe(Subscriber subscriber) {  
                Iterator<Integer> it = iter.iterator();  
  
                subscriber.onSubscribe(new Subscription() {  
                    @Override  
                    public void request(long n) { // 정보를 몇개나 받을 지?  
                        try {  
                            while (n-- > 0) {  
                                if (it.hasNext()) {  
                                    subscriber.onNext(it.next());  
                                } else {  
                                    subscriber.onComplete();  
                                    break;                                }  
                            }  
                        } catch (RuntimeException e) {  
                            subscriber.onError(e);  
                        }  
                    }  
  
                    @Override  
                    public void cancel() {  
  
                    }  
                });  
            }  
        };  
  
        Subscriber<Integer> sub = new Subscriber<Integer>() {  
            Subscription subscription;  
  
            @Override  
            public void onSubscribe(Flow.Subscription subscription) {  
                System.out.println("onSubscribe");  
                this.subscription = subscription;  
                this.subscription.request(1); //처음 받을 데이터 사이즈  
            }  
  
            // 다음 받을 것을 어떻게 할지? 처리할 수 있는지 등등  
            // 나중에 스케쥴러가 스케쥴링해준다.  
            @Override  
            public void onNext(Integer item) {  
                System.out.println("onNext = " + item);  
                this.subscription.request(1);  
            }  
  
            @Override  
            public void onError(Throwable throwable) {  
                System.out.println("onError");  
            }  
  
            @Override  
            public void onComplete() {  
                System.out.println("onComplete");  
            }  
        };  
  
        pub.subscribe(sub);  
    }  
}
```