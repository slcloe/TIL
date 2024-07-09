# Item 30 - 이왕이면 제네릭 메서드로 만들기 

#### 제네릭 메서드 작성법
```java
// <E> 순서대로
// 1. 타입 매개변수 목록
// 2. 반환 타입
// 3. 파라미터 타입
public static <E> Set<E> union(Set<E> s1, Set<E> s2) { 
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```
* 경고 없이 컴파일 가능, 타입 안정, 쓰기도 쉽다.


#### 제네릭 싱글톤 팩터리
* 제네릭은 런타임 시에 타입 정보가 소거 되기 때문에 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.
	* 불변 객체를 여러 타입으로 이용할 수 있다.
	* 항등함수를 만들 수 있다.
		> **항등 함수란?**
		> 입력 값을 수정 없이 그래도 반환하는 특별한 함수
1. 불변 객체
```java
public static final <T> List<T> list() {
	return (List<T>) EMPTY_LIST;
}
```

2. 항등 함수 
* 항등 함수 구현
```java
private static final UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

* 항등 함수 사용 예시
```java
public static void main(String[] args) {
    String[] strings = { "삼베", "대마", "나일론" };
    UnaryOperator<String> sameString = identityFunction();
    for (String s : strings) {
        System.out.println(sameString.apply(s));
    }

    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number n : numbers) {
        System.out.println(sameNumber.apply(n));
    }
}
```
#### 재귀적 타입 한정
* 타입 매개변수의 범위를 한정하는 것이다.
```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```	
* Comparable 을 구현한 타입만 가능하다.



<!-- 
```java

```
 -->