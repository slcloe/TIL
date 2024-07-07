# Item 11 - equals를 재정의 할 때 hashCode도 재정의하기

* HashMap, HashSet 같은 컬렉션 원소로 사용하기 위해서 hashCode를 재정의 해야한다.

> 1. 두 객체가 equals에 의해 같다고 판단했다면, 앱이 실행되는 동안 일관되게 같은 값을 반환해야한다.
> 2. equals로 같다고 판단했다면, 두 객체는 hashCode값도 같아야한다.
> 3. equals로 다르다고 판단했더라도, 두 객체의 hashCode는 같을 수는 있다.

###### 만약 적합하지 않은 해시코드를 구현했다면?
* 해쉬테이블이 O(1)이 아닌 O(N)으로 느려질 것이다. 

```java
// 해쉬코드 예시 1 - 성능 좋은 버전
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix); 
	result = 31 * result + Short.hashCode(lineNum); 
	return result;
}
```

```java
// 해쉬코드 예시 2 - 성능이 좀 아쉬운 버전
@Override public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode); 
}
```

```java
// 해쉬코드 예시 3 - 지연 초기화하는 버전 (스레드 안정성까지 고려)
private int hashCode; // 자동으로 0으로 초기화된다.
@Override public int hashCode() { 
	int result = hashCode;
	if (result == 0) {
		result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix); 
		result = 31 * result + Short.hashCode(lineNum); 
		hashCode = result;
	}
	return result; 
}
```

