# Item 83 - 지연 초기화는 신중히 사용하기

#### 지연 초기화
* 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다
* 사용
	* 정적 필드와 인스턴스 필드 모두 사용한다.
	* 주로 최적화 용도로 사용한다.
	* 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결한다.

#### 지연 초기화의 주의사항
* 인스턴스 생성 시의 초기화 비용은 줄지만, 처음 접근하는 비용은 커진다.
	* 따라서, 성능 비교가 필요하다.
* 멀티스레드 환경일 때,
	* 지연 초기화 필드를 둘 이상의 스레드가 공유한다면 반드시 동기화를 해야한다.

#### 지연 초기화의 필요성
* 인스턴스 중 사용률은 낮지만, 초기화하는 비용이 클 때
	* 하지만, 이 경우는 직접 성능을 체크해야한다.
* 대부분의 상황에서는 일반 초기화를 권장한다.

##### 인스턴스 필드를 초기화하는 일반적 방법
```java
private final FieldType field = computeFieldValue();
```

##### 인스턴스 필드의 지연 초기화 - synchronized 접근자 방식
```java
private FieldType field;

private synchronized FieldType getField() { 
	if (field == null)
		field = computeFieldValue(); 
	return field;
}
```
* synchronized 키워드 사용
	* 초기화 순환성을 막기 위해 사용함.

##### 정적 필드용 지연 초기화 홀더 클래스 관용구
```java
private static class FieldHolder {
	static final FieldType field = computeFieldValue(); 
}
private static FieldType getField() { return FieldHolder.field; }
```
* 지연 초기화 홀더 클래스 관용구
	* 클래스가 처음 쓰일 때 비로소 초기화된다는 특성
* getField 호출 -> FieldHolder.field 읽힘 -> FieldHolder 클래스 초기화
	* getField 접근 시 동기화를 전혀 하지 않기 때문에 성능 저하의 걱정이 없다.


#### 이중 검사 관용구
* 검사1. 동기화 없이 검사함. (필드가 아직 초기화 되지 않았을 때)
* 검사2. 동기화 후 검사함. (필드가 아직 초기화 되지 않았을 때)
	* 초기화 후로는 동기화 하지 않기 때문에 volatile로 선언한다.
```java
private volatile FieldType field;

private FieldType getField() {
	FieldType result = field;
	if (result != null) // 첫 번째 검사 (락 사용 안 함)
		return result;
	synchronized(this) {
		if (field == null) // 두 번째 검사 (락 사용)
			field = computeFieldValue(); 
		return field;
	} 
}
```
* 성능 때문에 인스턴스 필드를 지연 초기화 할 때 사용한다.
	* 첫 접근의 동기화 비용을 없애준다.

#### 단일 검사 관용구
* 아중 검사 관용구에서 두번째 검사를 생략한 경우다.
* 반복해서 초기화 해도 상관없는 필드가 포함되어 있을 경우 사용할 수 있다.
```java
private volatile FieldType field;
private FieldType getField() { 
	FieldType result = field; 
	if (result == null)
		field = result = computeFieldValue(); 
	return result;
}
```
* 필드 접근 속도를 높여주지만, 초기화가 스레드당 최대 한번 더 이루어질 수 있다.
<!--
```java

```
 -->