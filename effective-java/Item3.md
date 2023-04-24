# Item3 - 생성자나 열거 타입으로 싱글톤임을 보증하라.

## 첫번째 방법 : `private` 생성자 + `public static final` 필드

```java
public class Elvis{
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
}
```

* 장점
	* 간결하고, 싱글톤임을 API에 들어낼 수 있다. 
* 단점
	* 단점 1 : 인터페이스를 사용하지 않으면 싱글톤을 사용하는 클라이언트 테스트하기 어려워진다.
		* 인터페이스를 사용하면 `Mock`객체를 만들어서 테스트에 사용할 수 있다.
	* 단점 2 : 리플렉션으로 private 생성자를 호출할 수 있다.
		* 플래그 같은 것을 만들어서 2번째 생성되는 것을 막을 수 있다.
	* 단점 3 : 역직렬화 할 때 새로운 인스턴스가 생길 수 있다.
		* `readResolve`메소드를 만들어주면 `Override`가 아님에도 `Override`처럼 동작해서 기존에 있는 `Instance`를 줄 수 있다.

단점을 보완한 싱글톤
```java
public class Elvis implements IElvis,Serializable {
	public static final Elvis INSTANCE = new Elvis();
	private static boolean created;
	
	private Elvis() {
		if(created){
			throw new UnsopportedOperationException("");
		}
		created = true;
	}
	private Object readResolve(){
		return INSTANCE;
	}
}
```

## 두번째 방법 : `private`생성자 + 정적 팩터리 메서드

```java
public class Elvis{
	private static final Elvis INSTANCE = new Elvis();
	private Elvis(){ }
	public static Elvis getInstance(){ 
		return INSTANCE; 
	}
}
```

* 장점
	* 장점 1 : API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
		* `getInstace` 내부를 변경하면 싱글턴이 아니게 될 수 있음
	* 장점 2 : 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
		* 제네릭한 팩토리를 사용할 때 인스턴스는 동일하지만 각각의 원하는 타입으로 받을 수 있다.
		* `equals`를 사용하거나 `hashcode`를 보면 같은 인스턴스지만 다른 타입임으로 `==`으로 비교할 순 없다.
	* 장점 3 : 정적 팩터리의 메서드 참조를 공급자로 할 수 있다.
		* 공급자 = `FunctionalInterface`
		* 인자없는 메소드가 어떠한 인스턴스를 반환하는 형태일 때(`getInstance`) `Supplier`로 사용할 수 있다.
		* 메서드 참조 = `Elvis::getInstace`
* 단점 - 첫번째 방법과 동일하다.

## 세번째 방법 : 열거 타입

```java
public enum Elvis{
	INSTACNE;
}
```

가장 간결한 방법이며, 직렬화와 리플렉션에도 안전하다.  
대부분의 상황에는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
테스트 시 인터페이스를 구현하도록 해서 테스트도 간단하게 할 수 있다.

# 추가 개념

## 메소드 참조

메소드 하나만 호출하는 람다 표현식을 줄여쓰는 방법
* 스태틱 메소드 레퍼런스 `Class::mehod`
	* 클래스 레퍼런스를 통해서 참조 
* 인스턴스 메소드 레퍼런스 `instance::method`
	* 용법은 비슷하나 앞에 인스턴스를 한번 만들고 사용한다. 
* 임의 객체의 인스턴스 메소드 레퍼런스 `Class::method`
	* 첫번째 인자가 자기 자신
* 생성자 레퍼런스 `Class::new`
	* 다른 인자를 받는 생성자를 쓰고 싶은 경우 `Function` 인터페이스를 사용하면 된다.
	* `Function<LocalDate, Person> personFucntion = Person::new;`
* https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html

## 함수형 인터페이스

자바가 제공하는 기본 함수형 인터페이스
* 함수형 인터페이스는 람다 표현식과 메소드 참조에 대한 `타겟 타입`을 제공한다.
* 타겟 타입은 변수 할당, 메소드 호출, 타입 변환에 활용할 수 있다.
* 자바에서 제공하는 기본 함수형 인터페이스 익혀 둘 것. (`java.util.function` 패지키)
	* `Function`
		* 2개의 제네릭 타입을 받는데 처음은 `input`타입 다음은 `output`타입이다.
	* `Supplier`
		* 받는 인자가 없고 `ouput`이 나오기만 하는 경우
	* `Consumer`
		* 받는 인자는 있지만 `return`이 없는 것
		* `System.out::println`
	* `Predicate`
		* 인자를 하나 받아서 `boolean`을 리턴하는 것
* 함수형 인터페이스를 만드는 방법
* 심화
	* `Understanding Java method invocation with invokedynamin`
	* `LamdaMetaFactory`

## 객체 직렬화

객체를 바이트스트림으로 상호 변환하는 기술
* 바이트스트림으로 변환한 객체를 파일로 저장하거나 네트워크를 통해 다름 시스템으로 전송할 수 있다.
* `Serializable` 인터페이스 구현
* `transient`를 사용해서 직렬화 하지 않을 필드 선언하기
* `serialVersionUID`는 언제 왜 사용하는가?
	* 명시적으로 선언하지 않으면 `runtime` 도중에 다시 만들어진다.
	* 클래스가 변경되면 새로운 값을 지정한다.
	* 명시적으로 선언하려면 `private static final long serialVersionUID` 으로 선언한다.
* 심화
	* `객체 직렬화 스팩`
	* `Externalizable`