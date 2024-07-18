# Item 62 - 다른 타입이 적절하다면 문자열 사용을 피하기

#### 문자열을 쓰지 말아야할 사례
1. 문자열을 다른 값 타입으로 사용할 경우
	* 적절한 값 타입이 있다면 그것을 사용해야한다. 
2. 열거타입을 문자열로 대체하는 경우
3. 혼합 타입을 문자열로 대신하는 경우
	* ex. `String compoundKey = className + "#" + i.next();`
	* 두 요소를 구분해주는 문자#이 두 요소 중 하나에 쓰이면 어떤 문제가 생길지 모른다.
	* 문자열 파싱과정에서 성능 악화가 일어난다.
	* 차라리 전용 클래스를 새로 만들면 쉽게 해결할 수 있다.
4. 권한을 문자열로 표현하는 경우
###### 문자열로 권한을 구분한 사례
```java
public class ThreadLocal {
private ThreadLocal() { } // 객체 생성 불가 // 현 스레드의 값을 키로 구분해 저장한다.
public static void set(String key, Object value);
// (키가 가리키는) 현 스레드의 값을 반환한다.
public static Object get(String key); }
```
* ThreadLocal 에서 쓰는 변수를 문자열 키로 구분했다.
* 이렇게 되면, 쓰레드 고유키가 전역 공간에 공유된다. -> 보안에 취약하다.
* 같은 변수를 공유할 수 있는 상황이 있을 수 있다.

###### Key 클래스로 권한 구분하기
```java
public class ThreadLocal {
	private ThreadLocal() { } // 객체 생성 불가 
	public static class Key { // (권한)
		Key() { } 
	}
	// 위조 불가능한 고유 키를 생성한다. 
	public static Key getKey() { return new Key(); }
	public static void set(Key key, Object value);
	public static Object get(Key key); 
}
```
* 이전과는 다르게 Key 클래스를 사용하여 권한을 구분했다.
* 하지만 Key는 스레드 지역변수 자체가 되어버렸다.

###### 그래서 Key 대신 ThreadLocal 로 리팩토링을 진행했다.
```java
public final class ThreadLocal { 
	public ThreadLocal();
	public void set(Object value); 
	public Object get();
}
```

###### 매개변수화
```java
public final class ThreadLocal<T> { 
	public ThreadLocal();
	public void set(T value); 
	public T get();
}
```
* 매개변수화를 통해 타입 안정성을 제공한다.



<!--
```java
public final class ThreadLocal<T> { public ThreadLocal();
public void set(T value); public T get();
}
```
 -->