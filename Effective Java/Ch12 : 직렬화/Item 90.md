# Item 90 - 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하기

#### 직렬화 프록시 패턴 설계
1. 중첩 클래스 설계 + private static 으로 선언
	* 이때, 중첩클래스가 바깥 클래스의 직렬화 프록시다.
2. 중첩 클래스의 생성자는 1개, 바깥 클래스를 매개변수로 받아야한다.
3. 일관성 검사나 방어적 복사는 필요 없다.
4. 중첩 클래스(직렬화 프록시) 와 바깥 클래스는 모두 Serializable을 구현해야한다.

* 직렬화 프록시 예제
```java
private static class SerializationProxy implements Serializable { 
	private final Date start;
	private final Date end;

	SerializationProxy(Period p) { 
		this.start = p.start;
		this.end = p.end; 
	}

	private static final long serialVersionUID = 234098243823485285L; // 아무 값이나 상관없다. (아이템 87)
}
// 직렬화 프록시 패턴용 writeReplace 메서드 
private Object writeReplace() {
	return new SerializationProxy(this); 
}

// 직렬화 프록시 패턴용 readObject 메서드
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
	throw new InvalidObjectException("프록시가 필요합니다.");
}

// Period.SerializationProxy용 readResolve 메서드 
private Object readResolve() {
	return new Period(start, end); // public 생성자를 사용한다. 
}
```
* writeReplace
	* 직렬화가 다뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다.
	* 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다는 뜻이다.
* readObject
	* 불변식 훼손을 방지할 수 있다.
* readResolve
	* 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 한다.
	* 장점. 공개 API으로만 구성되어 있다. -> 불변식 검사를 따로 하지 않아도 된다.

#### 직렬화 프록시의 장점
1. 가짜 바이트 스트림 공격 & 내부 필트 탈취 공격을 프록시 수준에서 차단해준다.
2. 직렬화 프록시는 Period의 필드를 final로 선언해도 된다.
	* 불변 객체로 만들 수 있다.
	* 역질렬화 시 유효성 검사를 따로 하지 않아도 된다.

#### EnumSet의 직렬화 프록시

```java
private static class SerializationProxy <E extends Enum<E>> implements Serializable {
// 이 EnumSet의 원소 타입
private final Class<E> elementType;
// 이 EnumSet 안의 원소들
private final Enum<?>[] elements;
SerializationProxy(EnumSet<E> set) { elementType = set.elementType;
elements = set.toArray(new Enum<?>[0]); }
private Object readResolve() {
EnumSet<E> result = EnumSet.noneOf(elementType); for (Enum<?> e : elements)
result.add((E)e); return result;
}
private static final long serialVersionUID = 362491234563181265L;
}
```
* 정적 팩터리를 통해 객체를 생성한다.
* 열거 타입 원소가 64개 이하면 RegularEnumSet을, 그보다 크면 JumboEnumSet을 사용한다.
* 직렬화시에도 동일하게 동작한다.
* 직렬화 프록시 패턴 한계
	1. 확장할 수 있는 클래스에는 적용할 수 없다.
	2. 객체 그래프에 사이클이 있는 클래스는 적용할 수 없다.
		* readResolve안에서 호출하면 ClassCastException이 발생한다.
		(직렬화 프록시만 구현됐지 실제 객체는 아직 만들어진 것이 아니기 때문이다.)
	3. 방어적 복사보다 성능 저하가 생긴다.
<!--
```java

```
 -->