# Item 89 - 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하기

#### 싱글톤에서의 직렬화
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis(); 
	private Elvis() { ... }
	public void leaveTheBuilding() { ... } 
}
```
* 이 코드에서 직렬화를 구현하면 더이상 싱글톤이라고 할 수 없다.
* 따라서, readSolve를 통해 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다.
	* 이때, 생성된 새로운 객체는 참조를 유지하지 않기 때문에 GC 대상이 된다.

```java
// 인스턴스 통제를 위한 readResolve - 개선의 여지가 있다! 
private Object readResolve() {
	// 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
	return INSTANCE; 
}
```
* 역직렬화된 객체는 무시하고 클래스 초기화 때 만들어진 인스턴스를 반환한다.
* 따라서 인스턴스의 필드를 모두 transient로 선언하자.


##### 잘못된 싱글톤 예시
```java
public class Elvis implements Serializable {
	public static final Elvis INSTANCE = new Elvis(); 
	private Elvis() { }

	private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

	public void printFavorites() {
		System.out.println(Arrays.toString(favoriteSongs)); 
	}

	private Object readResolve() {
		return INSTANCE; 
	}
}
```
* transient가 아닌 참조 필드가 있다면, 
	* 필드의 원래 타입에 맞는 값을 반환하려고 한다.

```java
public class ElvisStealer implements Serializable { 
	static Elvis impersonator;
	private Elvis payload;
	private Object readResolve() {
		// resolve되기 전의 Elvis 인스턴스의 참조를 저장한다. 
		impersonator = payload;

		// favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
		return new String[] { "A Fool Such as I" }; 
	}
	private static final long serialVersionUID = 0; 
}
```
* 싱글톤이 역직렬화될 때 Stealer의 readResolve 메서드가 먼저 호출된다.
* 이때, 역직렬화 과정 중에 있는 싱글턴의 참조가 Stealer의 인스턴스 필드에 담긴다.
  


##### 싱글톤 객체가 2개가 생성된다!
```java
public class ElvisImpersonator {
	// 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림! 
	private static final byte[] serializedForm = {
		(byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
		0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6, 
		(byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
		0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
		0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
		0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c, 
		0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74, 
		0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76, 
		0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00, 
		0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01, 
		0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64, 
		0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b, 
		0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
	};
	public static void main(String[] args) {
		// ElvisStealer.impersonator를 초기화한 다음,
		// 진짜 Elvis(즉, Elvis.INSTANCE)를 반환한다.
		Elvis elvis = (Elvis) deserialize(serializedForm); Elvis impersonator = ElvisStealer.impersonator;
		elvis.printFavorites();
		impersonator.printFavorites(); 
	}
}

```
* 이 코드를 실행시키면 서로 다른 2개의 인스턴스를 생성할 수 있다.

##### 열거 타입 싱글톤
```java
public enum Elvis { 
	INSTANCE;
	private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
	public void printFavorites() {
		System.out.println(Arrays.toString(favoriteSongs)); 
	}
}
```

#### readResolve 메서드의 접근성
* final 클래스 : readResolve는 private로 선언
* final 이 아닌 클래스
	1. private 로 선언 시
		* 하위 클래스에서 사용할 수 없다.
	2. packaage-private로 선언 시
		* 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.
	3. protected, public으로 선언 시
		* 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.
	4. protected, public & 하위 클래스 재정의 X
		* 하위 클래스의 인스턴스를 역직렬화하면, 상위 클래스의 인스턴스를 생성하기 때문에 ClassCastException을 던진다.
<!--
```java

```
 -->