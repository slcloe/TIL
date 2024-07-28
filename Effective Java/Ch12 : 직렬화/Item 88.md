# Item 88 - readObject 메서드는 방어적으로 작성하기

#### 직렬화 시, 불변식 보장
* readObject 메서드 
	* public 생성자와 같은 역할을 한다. (매개변수로 바이트 스트림을 받는 생성자)
		1. 인수 유효성 검사
		2. 매개변수 방어적 복사
		3. 재정의 가능 메서드를 호출해서는 안된다.(Item 19)
	* 위 과정을 하지 않으면, 허용되지 않은 인스턴스를 만들 수 있다.
* 직렬화 프록시 패턴 사용


#### 역직렬화된 객체의 유효성 검사
* readObject.defaultReadObject()를 호출해 역직렬화된 객체의 유효성을 검사해야한다.
	* throw InvalidObjectException을 던져 오류를 확인하자.
* 이를 통해 불변식을 가지는 인스턴스를 수정할 수 없게 된다.
* 역질렬화 할 때는 불변식을 가지는 필드를 모두 방어적으로 복사해서 사용하자.


#### 방어적 복사와 유효성 검사를 수행하는 readObject메서드
```java
private void readObject(ObjectInputStream s)
	throws IOException, ClassNotFoundException {

	s.defaultReadObject();
	
	// 가변 요소들을 방어적으로 복사한다.
	start = new Date(start.getTime()); 
	end = new Date(end.getTime());

	// 불변식을 만족하는지 검사한다.
	if (start.compareTo(end) > 0)
		throw new InvalidObjectException(start +" after "+ end); 
}
```
* final 필드는 방어적 복사가 불가능하다.
	* 하지만 공격 위험 노출보다 낫다.



<!--
```java

```
 -->