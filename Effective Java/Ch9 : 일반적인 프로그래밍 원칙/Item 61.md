# Item 61 - 박상된 기본 타입보다는 기본 타입을 사용하기

##### 기본타입
* int, double, boolean

#### 박싱된 기본 타입
* Integer, Double, Boolean

#### 기본 타입과 박싱타입의 차이
1. 기본 타입은 값만 가지고 있지만, 박싱된 기본 타입은 값과 식별성 속성을 갖는다.
2. 기본 타입값은 언제나 유효하지만, 박싱된 기본 타입은 null 과 같은 유효하지 않은 값을 가질 수 있다.
3. 기본 타입이 박싱 타입보다 시간과 메모리 사용면에서 훨씬 효율적이다.


#### 박싱 타입의 오류
1. 박싱 타입에 == 연산자를 사용하면 오류가 일어난다.
```java
Comparator<Integer> naturalOrder =
		(i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```
* new Integer(42) - new Integer(42) 를 해당 연산자로 비교했을 때, 
	* (i == j)연산에서 두 객체의 값 비교가 아닌, `객체참조`의 식별성을 비교한다.
	* 따라서 정적 compare 메서드를 사용해야한다.

2. 기본 타입과 박싱 타입을 혼용한 연산에서는 박싱된 기본 타입이 기본 타입으로 형변환된다.
```java
public class Unbelievable {
	static Integer i;
	public static void main(String[] args) { 
		if (i == 42)
			System.out.println("String"); 
	}
}
```
* if 수행 시 NullPointerException을 던진다.
* null 참조를 언박싱하여 NPE가 발생한 것이다.

3. 성능 이슈

```java
public static void main(String[] args) { 
	Long sum = 0L;
	for (long i = 0; i <= Integer.MAX_VALUE; i++) {
		sum += i; 
	}
	System.out.println(sum); 
}
```
* sum 이 박싱타입이기 때문에 박싱과 언박싱이 반복해서 일어나 성능이 매우 저하된다.
* i 가 더해지고 그 값이 Long 으로 박싱된다.

#### 박싱타입이 필요할 때
1. 컬렉션의 원소, 키, 값으로 쓰일 때
	* 컬렉션에는 기본 타입을 담을 수 없기 때문이다.
2. 리플렉션을 통해 메서드를 호출할 때
<!--
```java
```
 -->