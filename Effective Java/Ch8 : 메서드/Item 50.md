# Item 50 - 적시에 방어적 복사본을 만들기

* 자바는 C / C++에 비해 안전한 언어라고 할 수 있다.
	* 네이티브 메서드를 사용하지 않으니까!
	* C/C++은 버퍼 오버런, 배열 오버런, 와일드 포인터와 같은 메모리 충돌 오류가 종종 일어난다.
	* 즉, 자바로 작성한 클래스는 시스템의 다른 부분과는 상관없이 불변식이 지켜진다.!

**하지만, 어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하게 해야한다.**

#### 불변식을 지키지 못한 코드
```java
public final class Period { private final Date start; private final Date end;
/**
* @param start 시작 시각
* @param end 종료 시각. 시작 시각보다 뒤여야 한다.
* @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
* @throws NullPointerException start나 end가 null이면 발생한다. */
public Period(Date start, Date end) { if (start.compareTo(end) > 0)
throw new IllegalArgumentException( start + " after " + end);
this.start = start;
this.end = end; }
public Date start() {
return start; }
public Date end() {
return end; }
... // 나머지 코드 생략 
}

Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); end.setYear(78); // p의 내부를 수정했다!
```
* private final 로 선언했지만, Date는 가변객체임을 몰랐기 때문에 문제가 생겼다.
* 자바8 이후로는 LocalDateTime, ZoneDateTime과 같은 불변 날짜 객체를 사용하면 된다.
* Date 는 옛날 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안된다.

#### 매개변수의 방어적 복사본을 만들기
```java
public Period(Date start, Date end) { this.start = new Date(start.getTime()); this.end = new Date(end.getTime());
if (this.start.compareTo(this.end) > 0) throw new IllegalArgumentException(
	this.start + " after " + this.end);
 }
```
* 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만든 후 이 복사본으로 유효성을 검사했다.
	* 멀티스레딩 환경일 때, 원본 객체의 유효성을 검사 후 복사하기 되면 그 사이에 다른 스레드에 의해 객체가 수정될 수 있기 때문이다.

#### 필드의 방어적 복사본을 반환하기
```java
// 공격
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); 
p.end().setYear(78); // p의 내부를 변경했다!

//방어
public Date start() {
	return new Date(start.getTime()); 
}
public Date end() {
	return new Date(end.getTime()); 
}
```
* 위 공격을 막기 위해 가변 필드의 방어적 복사본을 반환했다.

#### 방어적 복사와 clone()
* 생성자에서, 방어적 복사본을 만들 때 clone 함수를 쓰는 것은 위험하다.
	* clone함수가 해당 객체에서 정의하지 않은 함수일 수 있기 때문이다.
	* 매개변수가 제3자에 의해 확장될 수 있는 타입이라면, 방어적 복사본을 만들 때 clone함수를 사용해서는 안된다.
* 접근자 메서드에서, 방어적 복사본을 만들 때는 clone 함수를 사용해도 된다.
	* new Date() 같은 경우 java.util.Date가 확실하고, 신뢰할 수 있는 하위클래스이기 때문이다.
	* 인스턴스를 복사할 때는, 생성자나 정적 팩터리를 쓰는 것이 바람직하다.
* 가변인 내부 객체를 반환할 때는 방어적 복사본을 반환해야한다.
	* 길이가 1 이상인 배열은 무조건 가변이다.

#### 방어적 복사를 생략해도 될 때
* 호출자가 컴포넌트 내부를 수정하지 않을 것이라는 확신이 있을 때
	* 하지만, 이런 상황에도 호출자에서 값을 수정하지 말아야함은 명확히 문서화 하자!
* 객체의 통제권을 명백히 이전할 때
* 클래스와 클라이언트가 상호 신뢰할 수 있을 때
	* 래퍼클래스



<!--
```java

```
 -->