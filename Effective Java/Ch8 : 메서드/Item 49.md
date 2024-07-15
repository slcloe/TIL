# Item 49 - 매개변수가 유효한지 검사하기

### "오류는 가능한 한 빨리 잡아야 한다."
* 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워 지고 감지하더라도 오류의 발생 지점을 찾기 어려워진다.

* 메서드가 실행되기 전 매개변수를 확인하자!
	* 그러면 매개변수가 잘못 넘어왔을 때 바로 예외를 던질 수 있다.

#### 만약, 매개변수 검사를 하지 않는다면
1. 메서드가 수행되는 중간에 모호한 예외를 던지며 실패한다.
2. 메서드가 잘 수행되지만 잘못된 결과를 반환한다. ( 더 최악이다. )
3. 객체의 상태를 변화시켜 미래에 알 수 없는 시점에서 메서드와는 관련 없는 오류를 낸다.
	* 실패 원자성을 어기는 결과를 낳는다.
	> **실패 원자성이란?**
	> 유효성 검사에 실패한 이후, 객체의 상태가 영구적으로 바뀌어 다음 실행 결과가 다른 경우를 의미한다.
	> 메서드를 여러번 실패했을 때 매번 같은 결과를 반환했을 때, 실패원자성을 지켰다라고 말할 수 있다.


#### 매개변수의 문서화
1. public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야한다.
	* @throw 자바독 태그를 사용하기!
	* 보통은, IllegalArgumentException, IndexOutOfBoundsException, NullPointer Exception 을 사용한다.
2. 매개변수 제약과 그 제약을 어겼을 때 발생하는 예외도 문서화 해야한다.

#### 예외 문서화
```java
/**
* (현재 값 mod m) 값을 반환한다. 이 메서드는
* 항상 음이 아닌 BigInteger를 반환하다는 점에서 remainder 메서드와 다르다.
* @param m 계수(양수여야 한다.)
* @return 현재 값 mod m
* @throws ArithmeticException m이 0보다 작거나 같으면 발생한다. */
public BigInteger mod(BigInteger m) {
	if (m.signum() <= 0)
	throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
	... // 계산 수행
}
```
* @†hrow를 사용해 0보다 작거나 같은 값의 나머지를 구하면, ArithmeticException이 발생함을 알 수 있다.
* 하지만 m 이 null일 때, NullPointerException을 던진다는 설명은 없다.
	* @Nullable이나 이와 비슷한 애너테이션을 사용할 수 있지만, 이 설명은 클래스 수준에서 기술 했기 때문에 따로 적어주지 않았다.

#### 자바의 null 검사
```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```
* Objects.requireNonNull() : null이 매개변수로 들어오면 NullPointerException을 던진다.
* 반환값은 무시하고 필요한 곳 어디든 순수하게 null 검사 목적으로 사용 가능

#### assert를 사용한 매개변수 유효성 검증
```java
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >= 0 && length <= a.length - offset;
	... // 계산 수행
}
```
* 오직 유효한 값만이 메서드에 넘겨지는 것을 보증함.
* 실패하면 AssertionError를 던진다.
* 런타임에 아무런 효과도, 성능저하도 없다.
	* 하지만 java실행 시 cmd로 -ea, --enableassertions 플래그를 설정하면 런타임에 영향을 준다.
* public이 아닌 메서드라면 assert를 사용해 매개변수 유효성을 감증하자!

### "지금 당장 사용하지 않는 매개변수의 유효성도 검사해야한다."
* 지금 사용하지 않아서 매개변수의 유효성을 검증하지 않았지만, 나중에 NullPointerException이 일어나게 되면 디버깅이 힘들어진다.
* 따라서, 지금 사용하지 않는 매개변수의 유효성도 검사해야한다.

#### 매개변수의 유효성 검사를 하지 말아야할때,
1. 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
2. 계산 과정에서 암묵적으로 검사가 수행될 때
**예시**
* Collection.sort(list)의 값 비교 과정에서 서로 비교할 수 없는 타입이 들어가 있다면 ClassCastException을 던진다.

</br>

* 메서드는 최대한 범용적으로 설계해야한다.
* 매개변수 제약은 적을수록 좋다.

<!--
```java

```
 -->
