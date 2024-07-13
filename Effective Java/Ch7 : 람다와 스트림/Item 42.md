# Item 42 - 익명 클래스보다는 람다를 사용하기

#### 메서트 파라미터로 함수 객체를 받는 경우 : 익명 클래스 사용
```java
Collections.sort(words, new Comparator<String>() { 
	public int compare(String s1, String s2) {
	return Integer.compare(s1.length(), s2.length()); }
});
``` 
* 이 경우는 함수 코드가 너무 길어 함수형 프로그래밍에 적합하지 않다.

#### 메서드 파라미터로 함수 객체를 받는 경우 : 람다 사용
```java 
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
``` 
* 추상 메서드 하나 짜리 인터페이스(함수형 인터페이스)를 람다식을 사용해 만든다.
* 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자
	* ( 타입 추론 ) 컴파일러가 타입 추론을 할 때 필요한 정보 대부분을 제네릭에서 얻는다.

#### 메서드 파라미터로 함수 객체를 받는 경우 : 비교자 생성 메서드 활용
```java
Collections.sort(words, comparingInt(String::length));
words.sort(comparingInt(String::length));
``` 
##### 함수 객체를 저장해 상수별 동작을 구현한 열거 타입
```java
public enum Operation { 
	PLUS("+") {
		public double apply(double x, double y) { return x + y; } },
	MINUS("-") {
		public double apply(double x, double y) { return x - y; } },
	TIMES("*") {
		public double apply(double x, double y) { return x * y; } },
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; } 
	};
	private final String symbol;
	Operation(String symbol) { this.symbol = symbol; }
	@Override public String toString() { return symbol; }
	public abstract double apply(double x, double y); 
}

public enum Operation {
	PLUS ("+", (x, y) -> x + y), 
	MINUS ("-", (x, y) -> x - y), 
	TIMES ("*", (x, y) -> x * y),
	DIVIDE("/", (x, y) -> x / y);
	private final String symbol;
	private final DoubleBinaryOperator op; // 인터페이스 ( double 타입 인수 2개를 받아 Double 타입 결과를 돌려준다.)
	Operation(String symbol, DoubleBinaryOperator op) { 
		this.symbol = symbol;
		this.op = op;
	}
	@Override public String toString() { return symbol; }
	public double apply(double x, double y) {
		return op.applyAsDouble(x, y); 
	}
}
``` 
* 람다를 사용하여 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다.


#### 람다의 주의점
* 람다는 이름이 없어 문서화도 못한다. -> 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많다면 주의해야한다.
* 열거 타입 생성자에 넘겨지는 인수들의 타입은 컴파일 타임에 추론된다. -> 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다. ( 인스턴스는 런타임에 만들어짐)
* this키워드
	* 람다의 this키워드는 바깥 인스턴스를 가리킨다.
	* 람다는 자기 자신을 참조 할 수 없다.
	* 반면, 익명클래스의 this 키워드는 자기 자신을 가리킨다(함수 객체가 자신을 참조해야 할 경우에 사용)
* 가상머신 별로 직렬화 형태가 다를 수 있기 때문에 주의해야한다. ( 익명 클래스의 인스터스 포함.)




<!-- 
```java

``` 
-->

