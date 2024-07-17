# Item 56 - 공개된 API 요소에는 항상 문서화 주석을 작성하기


#### Javadoc 이란?
	* 소스코드 파일에서 문서화 주석이라는 형태로 기술된 설명을 추려 API문서로 변환해준다.
	* @literal, @code, @implSpe 등이 있다.
	* 상속용 클래스는 무엇을 하는지 기술해야함 ( what)
	* 메서드를 호출하기 위한 전제조건, 사후조건 명시
	* 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다
	* 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다

##### 주석에 필수적인 요소
* API의 문서화를 할 때 모든 public 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야한다.
* 상속용 클래스는 무엇을 하는지 기술해야함 ( what)
* 메서드를 호출하기 위한 전제조건, 사후조건을 명시하자.
* @throws, @params, @return 은 기본적으로 명시해줘야한다.
* 한클래스안에 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안된다.

#### Javadoc 태그
```java
/**
* Returns the element at the specified position in this list.
*
* <p>This method is <i>not</i> guaranteed to run in constant
* time. In some implementations it may run in time proportional * to the element position.
*
* @param index index of element to return; must be non-negative and less than the size of this list
* @return the element at the specified position in this list 
* @throws  IndexOutOfBoundsException if the index is out of range ({@code index < 0 || index >= this.size()})
*/

/**
* Returns true if this collection is empty. *
* @implSpec
* This implementation returns {@code this.size() == 0}. *
* @return true if this collection is empty
*/
public boolean isEmpty() { ... }

* A geometric series converges if {@literal |r| < 1}.
```
* html 태그를 사용할 수 있다.
* @code
	* 태그로 감싼 내용을 코드용 폰트로 렌더링한다.
	* 태그 안의 내용에 포함된 html요소나 다른 자바독 태그는 무시된다. 
* @implSpec
	* 해당 메서드와 하위 클래스 사이의 계약을 설명한다.
	*  `-tag "implSpec:a:Implementation Require ments:" `을 켜주지 않으면 @impleSpec 태그를 무시한다.
* @literal
	* html 마크업이나 자바독 태그를 무시할 때 쓴다.

#### 제네릭 주석
* 모든 타입 매개변수에 주석을 달아야한다.
```java
/**
* An object that maps keys to values. A map cannot contain
* duplicate keys; each key can map to at most one value. 
*
* (Remainder omitted) 
*
* @param <K> the type of keys maintained by this map 
* @param <V> the type of mapped values
*/
public interface Map<K, V> { ... }
```
#### 열거타입 주석
* 상수들에도 모두 주석을 달아야한다.
```java
/**
* An instrument section of a symphony orchestra. */
public enum OrchestraSection {
	/** Woodwinds, such as flute, clarinet, and oboe. */
	 WOODWIND,
	/** Brass instruments, such as french horn and trumpet. */ 
	BRASS,
	/** Percussion instruments, such as timpani and cymbals. */ PERCUSSION,
	/** Stringed instruments, such as violin and cello. */
	STRING; 
}
```
#### 에너테이션 주석
* 멤버들에도 주석을 모두 달아야한다.
```java
/**
* Indicates that the annotated method is a test method that * must throw the designated exception to pass.
*/
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	/**
	* The exception that the annotated test method must throw * in order to pass. (The test is permitted to throw any
	* subtype of the type described by this class object.)
	*/
	Class<? extends Throwable> value(); 
}
```
##### 주석의 상속
* @inheritDoc 태그슬ㄹ 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다.
<!--
```java

```
 -->
