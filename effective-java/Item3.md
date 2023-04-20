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
	* 단점 1 : 싱글톤을 사용하는 클라이언트 테스트하기 어려워진다.
	* 단점 2 : 리플렉션으로 private 생성자를 호출할 수 있다.
	* 단점 3 : 역직렬화 할 때 새로운 인스턴스가 생길 수 있다.

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
	* 장점 2 : 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
	* 장점 3 : 정적 팩터리의 메서드 참조를 공급자로 할 수 있다.

## 세번째 방법 : 열거 타입

```java
public enum Elvis{
	INSTACNE;
}
```

가장 간결한 방법이며, 직렬화와 리플렉션에도 안전하다.  
대부분의 상황에는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

# 추가 개념

## 메소드 참조

메소드 하나만 호출하는 람다 표현식을 줄여쓰는 방법

* 스태틱 메소드 레퍼런스
* 인스턴스 메소드 레퍼런스
* 임의 객체의 인스턴스 메소드 레퍼런스
* 생성자 레퍼런스
* https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html

## 함수형 인터페이스

자바가 제공하는 기본 함수형 인터페이스

* 함수형 인터페이스는 람다 표현식과 메소드 참조에 대한 `타겟 타입`을 제공한다.
* 타겟 타입은 변수 할당, 메소드 호출, 타입 변환에 활용할 수 있다.
* 자바에서 제공하는 기본 함수형 인터페이스 익혀 둘 것. (`java.util.function` 패지키)
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
* 심화
	* `객체 직렬화 스팩`