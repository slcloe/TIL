# Item 53 - 가변인수는 신중히 사용하기

### 가변인수란?
* 명시한 타입의 인수를 0개 이상 동적으로 받을 수 있는 것이다.
* 가변인수 메서드를 호출하면,
	* 인수의 개수와 길이가 같은 배열을 생성
	* 이후, 인수들을 배열에 저장하여 가변인수 메서드에 건내준다.
* 인수 개수는 런타임에 생성된 배열의 길이로 알 수 있다.
* printf, 리플랙션에서 쓰인다.

* 잘못 구현한 예시
```java
static int min(int... args) { 
	if (args.length == 0)
		throw new IllegalArgumentException("인수가 1개 이상 필요합니다."); 
	int min = args[0];
	for (int i = 1; i < args.length; i++)
		if (args[i] < min) 
			min = args[i];
	return min; 
}
```
* 위 코드는 런타임에서 실패한다.
* for-each 문이 쓰이지 않아서 명료하지 않다.
---
* 잘 구현한 예시
```java
static int min(int firstArg, int... remainingArgs) { 
	int min = firstArg;
	for (int arg : remainingArgs)
		if (arg < min) 
			min = arg;
	return min; 
}
```
* 이전처럼 인수를 넣지 않으면 컴파일 타임에 알 수 있다.
* for-each 문을 써 명로하게 표현되었다.


#### 가변인수의 성능
* 가변인수 메서드가 호출될 때마다 배열을 새로 할당하여 복사하는 과정이 필요하다.
	* 이를 최적화 하고 싶다면?

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```
* 가변인수가 size가 3인 배열을 많이 쓴다면 위와 같은 패턴을 통해 최적화를 이뤄낼 수 있다.
* EnumSet 의 정적팩터리도 위 기법을 사용해 열거 타입 집합 생성 비용을 최소화하고 있다.

<!--
```java

```
 -->
