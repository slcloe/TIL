# Item 73 - 추상화 수준에 맞는 예외를 던지기

* 메서드가 저수준 예외를 처리하지 않는다면?
	* 내부 구현 방식이 노출되고 high level API를 오염시킨다.
	* 다음 릴리즈에서 구현방식이 바뀌면 예상치 못한 예외를 맞닥뜨리게 된다.

#### 예외 번역
* 상위 계측에서 저수준의 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꾸는 방식
```java
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
	// 추상화 수준에 맞게 번역한다.
	throw new HigherLevelException(...); 
}
```

#### 예외 연쇄
* 문제의 근본 원인인 저수준의 예외를 고수준 예외와 함께 보내는 방식
* 만약, 예외 번역을 구현할 때, 저수준의 예외가 필요하다면 예외 연쇄를 사용하자.
```java
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
	// 저수준 예외를 고수준 예외에 실어 보낸다.
	throw new HigherLevelException(cause); 
}
```
```java
class HigherLevelException extends Exception { 
	HigherLevelException(Throwable cause) {
		super(cause); 
	}
}
```
* 대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있다.
	* Throwable.initCause(), getCause()
* 하지만, 저수준의 메서드가 반드시 성공하도록, 예외가 발생하지 않도록 하는것이 최선이다.
	* 상위 메서드의 매개변수 값을 하위 메서드로 건네기 전 미리 검사하는 방법
	* 상위 계층에서 예외를 모두 처리해서 API 호출자까지 도달하지 않게하는 방법
		* 이 경우는 java.util.logging 같이 로깅을 통해 기록을 해두자!
<!--
```java

```
 -->