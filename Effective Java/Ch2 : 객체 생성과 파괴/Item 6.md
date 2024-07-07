# Item 6 - 불필요한 객체 생성 피하기

불필요한 객체 생성 피하는 방법
1) 생성자 대신 정적 팩터리 메서드 사용
2) 생성 비용이 비싼 객체를 캐싱하기
3) 오토박싱 피하기



##### 1. 생성자 대신 정적 팩터리 메서드 사용
* `생성자`는 호출할 때마다 새로운 객체를 생성하지만 `팩터리메서드`는 그렇지 않다.
	ex) Boolean(String) vs Boolean.valueOf(String)


##### 2. 생성 비용이 비싼 객체를 캐싱하기

```java
static boolean isRomanNumeral(String s) { 
	return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
		+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
* 정규표현식용 Pattern 인스턴스는 한 번 쓰고 GC대상이 됨.  
	또한, 정규표현식에 해당하는 `유한 상태 머신`을 만들어 인스턴스 생성 비용이 높다.

<!-- 유한 상태 머신이란? -->

```java
public class RomanNumerals {
	private static final Pattern ROMAN = Pattern.compile(
		"^(?=.)M*(C[MD]|D?C{0,3})"
		+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

	static boolean isRomanNumeral(String s) {
		return ROMAN.matcher(s).matches(); 
	}
}
```
* 따라서, 성능 개선을 위해 Pattern 인스턴스를 초기화 과정에서 직접 생성해 캐싱하기
* 이후 인스턴스 재사용


##### 3. 오토박싱 피하기
* 오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술
* 하지만, 그렇다고 해서 기본타입과 박싱된 기본 타입이 같진 않다.

```java
private static long sum() { 
	Long sum = 0L; // 불필요한 오토박싱
	for (long i = 0; i <= Integer.MAX_VALUE; i++) 
		sum += i;
	return sum; 
}
```
* Long으로 선언함으로써 Long sum의 연산이 될 때마다 불필요한 Long 인스턴스(약 231개)가 생성된다.  

=> 박싱된 기본 타입보다는 기본 타입을 사용하자!


---

* JVM성능이 좋기 때문에 프로그램의 명확성, 간결성, 기능을 위한 객체생성은 괜찮다.
* 하지만, 단순히 객체 생성을 피하고자 나만의 객체 풀을 만드는 것은 권장하지 않는다.
	=> 일반적으로 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량 증가와 성능저하를 일으키기 때문
<!-- 왜 ?  -->




