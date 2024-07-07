 # Item 14 - Comparable을 구현할지 고민하기

 ##### Comparable의 compareTo 특징
 1. 단순 동치성 비교 가능
 2. 순서 비교 가능
 3. 제네릭하다

* Comparable을 구현한 객체들의 배열은 `Arrays.sort(array)` 로 정렬할 수 있다.

#### compareTo 메서드의 일반 규약
* 객체보다 작으면 음수, 같으면 0, 크면 양수 반환
1. sgn(x.compareTo(y)) == -sgn(y. compareTo(x))
2. (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
3.  x.compareTo(y) == 0이면 sgn(x. compareTo(z)) == sgn(y.compareTo(z))이다.
4. (x.compareTo(y) == 0) == (x. equals(y))
* 컬랙션들은 동치성을 비교할 때 equals 대신 compareTo를 사용한다.

##### 예외: BigDecimal class
	new BigDecimal("1.0") vs new BigDecimal("1.00")
	equals : 다르다고 판단.
	compareTo : 같다고 판단.

<!-- 왜 그러는지 체크하기 -->

#### compareTo 의 특징
* comparable 은 제네릭 인터페이스다. -> compareTo의 인수 타입은 컴파일 시에 정해진다.
* null이 인수로 넣어 호출되면 -> throw NullPointerException  
* Comparable 을 구현하지 않은 필드나 표준이 아닌 순서로 비교할 때는 Comparator를 사용한다.

> 정수 기본 타입 필드를 비교할 때,
> 박싱된 기본 타입 클래스들의 정적 메서드인 `compare`을 사용하기
> < > 사용은 지양한다. - 오버플로우 위험.


#### compareTo 구현
* 숫자 기본 타입 비교자 생성 메서드
```java
// 쉽지만 약간의 성능 저하가 있음.
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);
	if (result == 0) {
		result = Short.compare(prefix, pn.prefix); 
		if (result == 0)
			result = Short.compare(lineNum, pn.lineNum); 
	}
	return result; 
}

```

```java
// 성능 개선 버전
private static final Comparator<PhoneNumber> COMPARATOR = 
	comparingInt((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn -> pn.prefix) 
		.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn); 
}
```

* 객체 참조용 비교자 생성 메서드
	* 위 경우처럼 comparing, thenComparing 메서드가 존재한다.


```java
// 추이성 위반 경우
// 오버플로우를 일으키거나 , 부동소수점 오류를 일으킬 수 있다.
static Comparator<Object> hashCodeOrder = new Comparator<>() { 
	public 	int compare(Object o1, Object o2) {
		return o1.hashCode() - o2.hashCode(); }
};
```

```java
// 정적 compare 메서드를 활용한 Comparator
static Comparator<Object> hashCodeOrder = new Comparator<>() { 
	public int compare(Object o1, Object o2) {
		return Integer.compare(o1.hashCode(), o2.hashCode()); 
	}
};
```

```java
// Comparator 생성 메서드를 활용한 Comparator
static Comparator<Object> hashCodeOrder =
	Comparator.comparingInt(o -> o.hashCode());
```