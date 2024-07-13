# Item 39 - 명명패턴보다 애너테이션을 사용하기

#### 명명패턴
* 메서드의 이름에 패턴을 정하고 (ex. test.. ) Refelction 등으로 해당 패턴에 대한 작업을 정의하는 패턴

**단점**
1. 오탈자로 인한 오류 검출이 어렵다.
2. 함수명, 파라미터명, 클래스명 중 어디에 쓰여질지 모른다.
3. 프로그램 요소를 매개변수로 전달할 방법이 없다.


#### 메서드 Annotation
##### 메타에너테이션
	* 에너테이션 선언에 다는 에너테이션을 메타 에너테이션이라 한다.
```java
@Retention(RetentionPolicy.RUNTIME) // @Test가 런타임에도 유지되어야 한다.
@Target(ElementType.METHOD) // @Test는 반드시 메서드 선언에만 사용되어야 한다.
public @interface Test {
}
``` 
```java
public class Sample {
	@Test public static void m1() { } // 성공해야 한다. public static void m2() { }
	@Test public static void m3() { // 실패해야 한다.
		throw new RuntimeException("실패"); }
	public static void m4() { }
	@Test public void m5() { } // 잘못 사용한 예: 정적 메서드가 아니다. 
	public static void m6() { }
	@Test public static void m7() { // 실패해야 한다.
	throw new RuntimeException("실패"); }
	public static void m8() { } 
}
``` 

```java
import java.lang.reflect.*;
public class RunTests {
	public static void main(String[] args) throws Exception {
		int tests = 0;
		int passed = 0;
		Class<?> testClass = Class.forName(args[0]);
		for (Method m : testClass.getDeclaredMethods()) {
			if (m.isAnnotationPresent(Test.class)) { 
				tests++;
				try { 
					m.invoke(null);
					passed++;
				} catch (InvocationTargetException wrappedExc) {
					Throwable exc = wrappedExc.getCause();
					System.out.println(m + " 실패: " + exc); } catch (Exception exc) {
					System.out.println("잘못 사용한 @Test: " + m); }
					} 
		}
		System.out.printf("성공: %d, 실패\: %d%n", passed, tests - passed);
	} 
}
``` 
* getDeclareMethods()를 통해 클래스의 메서드를 가져오고 ReflectionAPI를 통해 해당 클래스의 메서드를 불러온다.
* isAnnotationPresent()를 통해 해당 어노테이션이 붙어있는지 확인한다.
* invoke(null)을 통해
	* 메서드 호출에 성공하면 -> 정적 메서드 ( 인스턴스 화 전에 호출이 가능한 메서드)
	* 메서드 호출에 실패하면 -> NullPointerException 호출

* 실행 결과
```java
public static void Sample.m3() failed: RuntimeException: Boom 
Invalid @Test: public void Sample.m5()
public static void Sample.m7() failed: RuntimeException: Crash 
성공: 1, 실패: 3
``` 

#### 매개변수를 하나 받는 애너테이션 
```java
import java.lang.annotation.*;
/**
* 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션 */
@Retention(RetentionPolicy.RUNTIME) @Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable> value(); 
}
``` 
* value() 필드는 기본 파라미터 값을 의미한다.
* `? extends Throwable` Throwable을 상속하는 모든 클래스를 받는다.

```java
public class Sample2 {
	@ExceptionTest(ArithmeticException.class) // 파라미터 인자를 ArithmeticException 로 넘겼다.
	public static void m1() { // 성공해야 한다. 
		int i = 0;
		i = i / i; 
	}
	@ExceptionTest(ArithmeticException.class)
	public static void m2() { // 실패해야 한다. (다른 예외 발생) 
		int[] a = new int[0];
		int i = a[1]; 
	}
	@ExceptionTest(ArithmeticException.class)
	public static void m3() { } // 실패해야 한다. (예외가 발생하지 않음) 
}
``` 

```java
if (m.isAnnotationPresent(ExceptionTest.class)) { 
	tests++;
	try {
		m.invoke(null);
		System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m); 
	} 
	catch (InvocationTargetException wrappedEx) {
		Throwable exc = wrappedEx.getCause();
		Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
		if (excType.isInstance(exc)) { 
			passed++;
		} else {
			System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc); 
		}
	} catch (Exception exc) {
		System.out.println("잘못 사용한 @ExceptionTest: " + m); 
	}
}
``` 
* isAnnotationPresent(ExceptionTest.class)를 통해 @ExceptionTest 에너테이션이 달린 메서드를 찾는다.
* @ExceptionTest의 인자와 같은 예외가 일어났을 경우 passed + 1

* 실행결과

```java
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable>[] value(); 
}
``` 

#### 배열 매개변수를 받는 애너테이션
```java
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable>[] value(); 
}
``` 
* 매개변수를 여러개 받을 수 있다.

```java
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class }) // 인자가 2개로 주어졌다.
public static void doublyBad() { List<String> list = new ArrayList<>();
	// 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
	// NullPointerException을 던질 수 있다.
	list.addAll(5, null); 
}
```

```java
if (m.isAnnotationPresent(ExceptionTest.class)) { 
	tests++;
	try { 
		m.invoke(null);
		System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m); 
	} catch (Throwable wrappedExc) {
		Throwable exc = wrappedExc.getCause();
		int oldPassed = passed;
		Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
		for (Class<? extends Throwable> excType : excTypes) {
			if (excType.isInstance(exc)) { 
				passed++;
				break;
			}
		}
		if (passed == oldPassed)
			System.out.printf("테스트 %s 실패: %s %n", m, exc); 
	}
}
``` 
* 이전과 테스트 코드가 동일하지만, 이번 에너테이션은 매개변수를 배열로 받기 때문에 for-each 를 활용하여 exception 타입이 일치한지 확인한다.


#### @Repeatable을 이용한 애너테이션

```java
// Repeatable 애너테이션
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD) 
@Repeatable(ExceptionTestContainer.class) 
public @interface ExceptionTest {
	Class<? extends Throwable> value(); 
}
// 컨테이너 애너테이션 
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
	ExceptionTest[] value(); 
}
``` 
* Repeatable 에너테이션을 반환하는 Container 애너테이션을 하나 더 정의해야한다.
* Container 애너테이션을 정의할 때, 내부 애너테이션의 타입의 배열을 반환하는 value 메서드를 정의해야한다.
* Container 애터네이션을 정의할 때, @Retention 과 @Target을 명시해야 한다.

</br>
* 사용 예시

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class) 
public static void doublyBad() { ... }
``` 


```java
if (m.isAnnotationPresent(ExceptionTest.class)
	|| m.isAnnotationPresent(ExceptionTestContainer.class)) { 
		tests++;
	try { 
		m.invoke(null);
		System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m); 
	} catch (Throwable wrappedExc) {
		Throwable exc = wrappedExc.getCause(); int oldPassed = passed; ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class); 
		for (ExceptionTest excTest : excTests) {
			if (excTest.value().isInstance(exc)) { 
				passed++;
				break; 
			}
		}
		if (passed == oldPassed)
		System.out.printf("테스트 %s 실패: %s %n", m, exc); 
	}
}
``` 
* isAnnotaionPresent는 Repeatable & Container 애너테이션을 모두 구분하기 때문에 제대로 검사가 이루어지지 않을 수 있다.
	`m.isAnnotationPresent(ExceptionRepeatableTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)` 로 사용하면 된다.
* getAnnotationByType은 이 두개의 애너테이션을 구분하지 않고 정보를 다 가져온다.


=> 명명패턴 대신 애너테이션을 적극 활용하자.
<!-- 
```java

``` 
-->