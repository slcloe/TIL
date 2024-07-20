# Item 72 - 표준 예외를 사용하기

#### 표준 예외 재사용시 이점
1. API가 다른 사람이 익히고 사용하기 쉬워진다.
2. 코드의 가독성이 올라간다.
3. 예외 클래스 수가 적을수록 메모리 사용량이 줄고 클래스를 적재하는 시간도 적게 든다.

#### 표준 예외 종류
##### 1. IllegalArgumentException
* 허용하지 않는 값이 인수로 건네졌을 때
	* null은 NullPointerException 으로 처리
* ex. 반복 횟수를 지정하는 매개변수에 음수를 건넬 때
##### 2. IllegalStateException
* 객체가 메서드를 수행하기에 적절하지 않은 상태일 때
* ex. 제대로 초기화 되지 않은 객체를 사용하려 할 때
##### 3. NullPointerException
* null을 허용하지 않는 메서드에 null을 건넸을 때

##### 4. IndexOutOfBoundsException
* 인덱스가 범위를 넘어섰을 때
* ex. 어떤 시퀀스의 허용 범위를 넘는 값을 건낼 때
##### 5. ConcurrentModificationException
* 허용하지 않는 동시 수정이 발견되었을 때
* ex.단일 스레드에서 사용하려고 설계한 객체를 여러 스레드에서 동시에 수정하려 할 때
	* 이 예외는 문제가 생길 가능성을 알려주는 역할이다.
##### 6. UnsupportedOperationException
* 호출한 메서드를 지원하지 않을 때
* ex. add만 구현되어 있는 List 구현체에 remove 메서드를 호출할 때

#### 표준 예외를 재사용하지 말것
* Exception, RuntimeException, Throwable, Error은 직접 재사용하지 말자.
	* 다른 예외들의 상위 클래스이기 때문에, 예외에 대해 구체적인 정보를 전달하기 어렵고 안정적인 테스트가 불가하다.
* 예외는 직렬화 할 수 있다.
	* 필드가 직렬화가 가능한지 확인해야한다.
	* serialVersionUID를 명시해야한다.

#### 다른 예외를 재사용하는 상황
* 상황에 부합한다면 표준 예외를 사용해야하지만, 맥락에 부합한다면 재사용을 고려해보자.
	* ex. 복소수, 유리수를 다루는 객체를 작성할 때, ArithmeticExcpetion, NumberFormatException을 재사용할 수 있다.

<!--
```java

```
 -->