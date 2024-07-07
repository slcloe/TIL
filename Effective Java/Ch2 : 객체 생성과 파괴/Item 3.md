# Item 3 - private 생성자나 열거 타입으로 싱글톤임을 보증하기


#### 싱글톤이란?
	  인스턴스를 오직 하나만 생성할 수 있는 클래스 
	  ( ex. stateless 객체, 시스템 컴포넌트 )

	  하지만, 테스트 시 mock으로 구현해야해서 어렵다.


* 싱글톤 만드는 방법
	1) public static final 사용
	2) 정적 팩터리 방식의 싱글톤
	3) Enum 방식의 싱글톤


#### 1. public static final 필드 방식의 싱글톤

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}
}
```
* 코드가 간결하다.
* private 생성자는 Elvis.INSTANCE를 초기화할 때 딱 한번만 호출됨.
* 하지만 리플렉션 API인 AccessibleObject.setAccessible을 사용하면 private 생성자 호출 가능

#### 2. 정적 팩터리 방식의 싱글톤
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}
	public static Elvis getInstance() { return INSTANCE; }

	// 싱글톤임을 보장해주는 readResolve 메서드
	private Object readResolve() {
		// 진짜 Elvis를 반환, 가짜 Elvis는 GC에 맡긴다.
		return INSTANCE;
	}
}
```
* 유지보수의 장점 : API를 바꾸지 않고도 싱글톤을 없앨 수 있다.
* 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
* 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다.
* 위 방법도 리플렉션 예외는 똑같이 적용됨.

##### 하지만 싱글톤 클래스를 직렬화하면??
1. Serializable 구현
2. readResolve 메서드 필요


#### 3. ENUM 방식의 싱글톤

```java
public enum Elvis {
	INSTANCE;
}
```
* 훨씬 간결하다.
* 직렬화 상황 / 리플렉션 공격에서도 다른 인스턴스 생성을 막아준다.
* 단, 싱글톤이 Enum 외의 클래스를 상속하는 것은 불가능하다.





