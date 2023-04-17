# item1 - 생성자 대신 정적 팩터리 메서드를 고려하라.

무조건 사용하라는 의미가 아니다. `고려하라`라는 뜻이다.

## 장점 1

장점 1 - 이름을 가질 수 있다. (`객체 생성을 더 잘 표현할 수 있다.`)

```java 
public class Order{
	private boolean prime;
	private boolean urgent;
	private Product product;

	public Order(Product product, Boolean prime){
		this.product = product;
		this.prime = prime;
	}

	//컴파일 에러가 발생한다.
	public Order(Product product, Boolean urgent){
		this.product = product;
		this.urgent = urgent;
	}

	//이건 가능
	public Order(Boolean urgent,Product product){
		this.product = product;
		this.urgent = urgent;
	}
}
```

생성자에서 시그니처는 타입까지 보기 때문에 매개 변수의 이름과는 상관없이 같은 `Product, Boolean` 타입을 가지는 생성자 2개를 생성할 수 없다.

따라서, 만드려면 매개 변수의 순서를 바꿔서 만든다.

하지만 정적 팩터리 메서드를 만들면 우리가 표현하고자 하는 객체 생성을 더 잘 표현할 수 있다.

```java
public class Order{
	private boolean prime;
	private boolean urgent;
	private Product product;

	public static Order primeOrder(Product product) {  
	    Order order = new Order();  
	    order.prime = true;  
	    order.product = product;  
	  
	    return order;  
	}  
  
	public static Order urgentOrder(Product product) {  
	    Order order = new Order();  
	    order.urgent = true;  
	    order.product = product;  
	    return order;  
	}
}
```

## 장점 2

장점 2 - 생성할 떄마다 객체를 생성하는 것이 아니라 미리 생성된 객체를 사용함을 보장할 수 있다.(`Boolean`,`valueOf`)

```java
public class Settings {  
    private boolean useAutoSteering;  
    private boolean useABS;  
    private Difficulty difficulty;  
  
    private Settings() {}
  
    private static final Settings SETTINGS = new Settings();  

	//정적 팩토리 메서드
    public static Settings getInstance() {  
        return SETTINGS;  
    }  
}
```

생성자를 `public`하게 제공하면 인스턴스의 생성을 컨트롤할 수 없다.

### Flywight 패턴
* 객체를 가볍게 만들어 메모리 사용을 줄이는 패턴
* 자주 변하는 속성(또는 외적인 속성, `extrinsit`)과 변하지 않는 속성(또는 내적인 속성, `intrinsit`)을 분리하고 재사용하여 메모리 사용을 줄일 수 있다.

## 장점 3 & 장점 4

장점 3 - 인터페이스 혹은 하위 클래스로 반환이 가능(`인터페이스 기반 프레임워크`,`인터페이스에 정적 메소드`)
장점 4 - 매개 변수에 따라서 다른 인스턴스 반환 가능(`EnumSet`)

```java
public class HelloServiceFactory{

	public static HelloService of(String lang){
		if (lang.equals("ko")){
			return new KoreanHelloService();
		} else {
			return new EnglishHeeloService();
		}
	}
}
```

`Interface`를 반환함으로써 실제 구현체를 숨길 수 있다.

또한, 자바 8부터는 `Interface`에 `static method`를 만들 수 있기 때문에 굳이 정적 팩토리 메서드를 가지고 있는 `Factory Class`를 만들어서 사용하지 않아도 된다.

```java
 interface HelloService {
 
	String hello();
	
	static HelloService of(String lang){
		if (lang.equals("ko")){
			return new KoreanHelloService();
		} else {
			return new EnglishHeeloService();
		}
	}
}
```

## 장점 5
장점 5 - 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.(`서비스 제공자 프레임워크`)

```java
public class HelloServiceFactory {

	public static void main(String[] args){
		//어떤 임의의 구현체가 올지 모르는 상태
		ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);
		Optional<HelloService> helloServiceOptional = loader.findFirst();
		helloServiceOptional.ifPresent(h -> {
			System.out.println(h.hello());
		});

		//이렇게 사용 가능하다. 하지만 chinese hello service에 의존하게 된다.
		HelloService helloService = new ChineseHelloService();
		System.out.println(helloSerivce.hello());
	}
}
```

서비스 제공자 프레임워크의 자바 기본 구현체 -> `ServiceLoader`

> chinese-hello-service

![META-INF](../img/META-INF.png)
![File-content](../img/File_content.png)

어떠한 구현체가 올지 모르지만 인터페이스 기반으로 코딩을 할 경우가 생긴다. 이때, 엄청난 유연함을 준다. 

> `JDBC`는 `ServiceLoader`가 등장하기 전에 만들어져서 이를 사용하진 않는다.

`JDBC`를 예시로 들 수 있다. 어떠한 구현체를 사용할 지 몰라도 인터페이스 기반으로 동작만 하면 됨으로 이러한 경우에서 응용할 수 있을 것이다.

## 단점 1
단점 1 - 생성자를 `private`로 만들어야되서 상속을 허용하지 않는다.

상속보다는 `private` 필드로 기존의 클래스의 인스턴스를 참조(`컴포지션`)하게 함으로써 어느 정도 장점으로 볼 수 있다.
또한, 생성자를 열어주고 정적 팩토리 메서드도 열어주는 경우도 있다.
예시로 `List`가 있다.

``` java
List<String> name = new ArrayList<>();
List.of("kim","lee");
```

## 단점 2
단점 2 - 어떤 메서드가 인스턴스를 생성하는 메서드인지 알기 어렵다.

따라서 `API 문서를 잘 작성`하고, 메서드 이름을 널리 알려진 규약을 사용하는 것이 좋다.


# 추가 개념

## 열거 타입 - `Enumeration`
* 상수 목록을 담을 수 있는 데이터 타입
* 특정한 변수가 가질 수 있는 값을 제한할 수 있다. `Type-Safety`를 보장할 수 있다.
* `싱글톤 패턴`을 구현할 때 사용하기도 한다.
* `JVM`에서 오직 하나만 만들어 진다.

### `EnumMap`과 `HashMap`, `EnumSe`t과 `HashSet`의 차이
`EnumMap`은 `HashMap`과 유사하게 `Map`을 구현한 구현체이지만 `Key`를 `Enum`으로 가진다는 특성을 가지고 있다. 둘은 몇가지 차이점을 가지고 있다.  
`HashMap`의 경우 key를 bucket에 저장하고 bucket이 `linked list`를 참조하는 형태로 되어 있는 반면에 `EnumMap`의 경우 key에 들어올 값이 한정되어 있기 때문에 `Array`를 선언하고 해당 index에 값을 넣으면 된다.
`index`를 가지고 올 때는 `Enum의 ordinal 함수`를 사용해서 바로 순서를 가지고 올 수 있다.

`EnumSet`과 `HashSet`도 유사하지만, `HashSet`의 경우 내부 구현은 `HashMap`을 이용하여 Map의 value가 있다 없다를 표현하는 `지시자`가 들어가지만, `EnumSet`의 경우 값이 있다 없다면 표시하면 됨으로 `bitvector`를 사용해서 구현이 가능하다. (`010011...`)

이런 차이점으로 `Enum`을 `Key`로 사용하는 것에 몇가지 장점이 있다.

#### 1. 속도가 더 빠르다.
`Enum`은 `JVM`에서 단일 객체임을 보장하기 때문에 `Hashing`하는 과정이 필요하지 않다.
따라서, `HashMap`보다 성능이 더 좋다.

#### 2. 순서를 기억한다.
여기서 말하는 순서는 사용자가 입력한 순서가 아니라 `Enum`에 명시되어 있는 순서를 말한다. 이미 순서가 정해져 있기 때문에 `TreeMap`처럼 정렬된다 하더라도 성능이 우수하다.

## 플라이웨이트 패턴 - `Flyweight`

* 객체를 가볍게 만들어 메모리 사용을 줄이는 패턴
* 자주 변하는 속성(또는 외적인 속성, `extrinsit`)과 변하지 않는 속성(또는 내적인 속성, `intrinsit`)을 분리하고 재사용하여 메모리 사용을 줄일 수 있다.

## 인터페이스에 정적 메서드

* 기본 메소드(`default method`)와 정적 메소드를 가질 수 있다.
* 기본 메소드
	* 인터페이스에서 메소드 선언 뿐 아니라, 기본적인 구현체까지 제공 가능
	* 기존의 인터페이스를 구현하는 클래스에 새로운 기능을 추가 가능
* 정적 메소드
	* 자바 9부터 `private static 메소드`도 가질 수 있다.
	* 단, `private 필드`는 아직도 선언할 수 없다.

`private`한 필드가 필요할 경우에는 `private 생성자`를 사용하는 인스턴스화 불가 동반 클래스(모든 메서드가 `static`이며 인스턴스화를 사용하지 못하게 하는 클래스)를 사용하기도 한다.

```java
List<Integer> numbers = new ArrayList();  
numbers.add(100);  
numbers.add(20);  
numbers.add(44);  
numbers.add(3);  
  
Comparator<Integer> desc = (o1, o2) -> o2 - o1;  
  
numbers.sort(desc.reversed());
```

`Comparator`에 `reversed`라는 메소드가 `default`로 정의되어 있음으로 `Comparator`만 구현했는데 사용이 가능하다.

## 서비스 제공자 프레임워크
확장 가능한 애플리케이션을 만드는 방법
애플리케이션의 코드는 유지하면서 코드 외적인 것을 변경해서 작동을 변경시킬 수 있는가?

* 주요 구성 요소
	* 서비스 제공자 인터페이스 (`SPI`)와 서비스 제공자 (`서비스 구현체`)
	* 서비스 제공자 등록 API (서비스 인터페이스의 구현체를 `등록하는 방법`)
	* 서비스 접근 API (서비스의 클라이언트가 서비스 인터페이스의 인스턴스를 `가져올 때 사용하는 API`)

``` java
//서비스 제공자 등록 API
//혹은 META-INF에 들어있는 파일
@Configuration  
public class AppConfig {  
    @Bean  
    public HelloService helloService() {  
        return new ChineseHelloService();  
    }

	...
}

//서비스 접근 API
public class App {  
    public static void main(String[] args) {  
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);  
        HelloService helloService = applicationContext.getBean(HelloService.class);  
        System.out.println(helloService.hello());  
    }  
}

public class HelloServiceFactory {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {  
        ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);  
        Optional<HelloService> helloServiceOptional = loader.findFirst();  
        helloServiceOptional.ifPresent(h -> {  
            System.out.println(h.hello());  
        });
    }  
}
```

* 다양한 변형
	* 브릿지 패턴
		* 구체적인 것과 추상적인 것을 분리해서 그 사이에 다리를 놓는 것
	* 의존 객체 주입 프레임워크
	* `java.util.ServiceLoader`
			* https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html
			* https://docs.oracle.com/javase/tutorial/ext/basics/spi.html

## 리플렉션 - `reflection`

* 클래스로더를 통해 읽어온 클래스 정보(실체가 아닌 투영된 정보)를 사용하는 기술
* 리플렉션을 사용해 클래스를 읽어오거나, 인스턴스를 만들거나, 메소드를 실행하거나, 필드의 값을 가져오거나 변경하는 것이 가능하다.
* 언제사용할까?
	* 특정 어노테이션이 붙어있는 필드 또는 메소드 읽어오기(`JUnit`, `Spring`)
	* 특정 이름 패턴에 해당하는 메소드 목록 가져와 호출하기(`getter`, `setter`)
	* ...
* https://docs.oracle.com/javase/tutorial/reflect/

```java
public class HelloServiceFactory {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class<?> aClass = Class.forName("me.whiteship.hello.ChineseHelloService");  
        Constructor<?> constructor = aClass.getConstructor();  
        HelloService helloService = (HelloService) constructor.newInstance();  
        System.out.println(helloService.hello());  
    }  
}
```