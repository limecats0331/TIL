# Test Double에 대해서 알아보기
테스트하려는 객체와 연관된 객체를 사용하기가 어렵고 모호할 때 대신 해 줄 수 있는 객체를 `Test Double`이라 한다.
예를 들어, `Service layer`를 테스트한다고 할 때, `Repositoy layer`에서 데이터 베이스에 제대로 연결되어 있는지 혹은 다른 미들웨어가 잘 연결되어 있는 지는 관심사가 아니다.
이런 경우, 실제 관심사만 테스트하기 위해서 사용된다.

> 영화 촬영 시 위험한 역활을 대신하는 stunt double에서 비롯되었다고 한다.

## Test Double의 종류
크게  `Dummy`, `Fake`, `Stub`, `Spy`, `Mock`으로 나눈다.

### 1. `Dummy`
* 가장 기본적인 Test Double
* 인스턴스화 된 객체가 필요하지만 기능은 필요없는 경우에 사용됨
* `Dummy` 객체는 메서드가 호출되었을 때 정상 작동을 보장하지 않음
* 객체는 전달되지만 사용되지 않는 객체

실제로 구현체를 필요하지만, 특정 테스트에서는 해당 구현체의 동작이 전혀 필요하지 않을 수 있다.
이처럼 `동작하지 않아도` 테스트에는 영향을 미치지 않는 객체를 `Dummy` 객체라고 한다.

### 2. `Fake`
* 복잡한 로직이나 객체 내부에서 필요로 하는 다른 외부 객체들의 동작을 단순화하여 구현한 객체
* 동작의 구현을 가지고 있지만 실제 비지니스에는 적합하지 않는 객체

`동작은 하지만` 실제 상용되는 객체처럼 정교하게 동작하지는 않는 개체. 
예를 들면, 테스트해야 하는 객체가 실제로 데이터베이스에 연결되어야 하지만 가짜 데이터베이스에 연결된 것 같은 동작을 하는 `Fake` 객체를 만들어서 테스트에 사용할 수 있다.
`Dummy`와 다르게 동일한 동작을 하도록 만들어 사용하는 객체가 `Fake`객체이다.

### 3. `Stub`
* `Dummy` 객체가 실제로 동작하는 것처럼 보이게 만들어 놓은 객체
* 인터페이스 또는 기본 클래스가 최소한으로 구현된 상태
* 테스트에서 호출된 요청에 대해 미리 준비해둔 결과를 제공

테스트를 위해 프로그래밍된 내용에 대해서만 `준비된 결과`를 제공하는 객체.

### 4. `Spy`
* 실제 객체처럼 동작시킬 수도 있고, 필요한 부분에 대해서는 `Stub`로 만들어서 동작을 지정할 수 있다.
* `Mockito`의 `@Spy` 를 사용할 수 있다.

### 5. `Mock`
* 호출에 대한 기대를 명세하고 내용에 따라 동작하도록 프로그래밍 된 객체이다.

`Stub`와의 차이는 `Stub`은 상태 검증(`state verification`)이며, `Mock`은 행위 검증(`behavior verification`)이라는 것이다.
상태 검증은 해당 로직이 동작했을 때 협력되는 `객체의 상태를 검증`함으로써 검증하는 것이다.
행위 검증은 검증하는 메소드가 참조하고 있는 메소드를 제대로 부르고 있는지에 대한 `행위를 검증`하는 것이다.

#### 참고 링크
[SpringBoot @MockBean, @SpyBean 소개](https://jojoldu.tistory.com/226)
[Mockito features in Korean](https://github.com/mockito/mockito/wiki/Mockito-features-in-Korean)
[Mockito @Mock @MockBean @Spy @SpyBean 차이점](https://cobbybb.tistory.com/16)
[[tdd] 상태검증과 행위검증, stub과 mock 차이](https://joont92.github.io/tdd/%EC%83%81%ED%83%9C%EA%B2%80%EC%A6%9D%EA%B3%BC-%ED%96%89%EC%9C%84%EA%B2%80%EC%A6%9D-stub%EA%B3%BC-mock-%EC%B0%A8%EC%9D%B4/)
