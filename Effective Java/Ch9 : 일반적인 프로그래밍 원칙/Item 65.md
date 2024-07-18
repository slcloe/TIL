# Item 65 - 리플렉션보다는 인터페이스를 사용하기

#### 리플렉션의 기능
1. 클래스의 생성자, 메서드, 필드 인스턴스를 가져올 수 있다.
2. 가져온 인스턴스를 통해 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수 있다.
	* ex. Method.invoke()는 어떤 클래스의 객체가 가진 임의의 메서드를 호출할 수 있게 한다.
3. 코드분석도구, 의존관계 주입 FW 처럼 리플렉션을 써야하는 앱들이 있다.

#### 리플렉션의 단점
1. 타입 검사의 이점을 가질 수 없다.
	* 오류를 런타임에 알게 된다.
2. 코드가 지저분해진다.
3. 성능이 덜어진다.
	* 일반 호출보다 리플렉션을 통한 메서드 호출이 훨씬 느리다.

> 따라서,
> 리플렉션은 제한된 형태로 써서, 단점을 피하고 이점을 취하자!
> 인스턴스 생성에만 쓰고, 그 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.

##### 리플렉션으로 생성하고 인터페이스로 참조해 활용하기
```java
public static void main(String[] args) {
	// 클래스 이름을 Class 객체로 변환
	Class<? extends Set<String>> cl = null; 

	try {
		cl = (Class<? extends Set<String>>) 
		// 비검사 형변환! Class.forName(args[0]);
	} catch (ClassNotFoundException e) {
		fatalError("클래스를 찾을 수 없습니다."); 
	}

	// 생성자를 얻는다.
	Constructor<? extends Set<String>> cons = null; 
	try {
		cons = cl.getDeclaredConstructor();
	}  catch (NoSuchMethodException e) {
		fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
	}

	// 집합의 인스턴스를 만든다. 
	Set<String> s = null; 
	try {
		s = cons.newInstance();
	} catch (IllegalAccessException e) {
		fatalError("생성자에 접근할 수 없습니다.");
	} catch (InstantiationException e) {
		fatalError("클래스를 인스턴스화할 수 없습니다.");
	} catch (InvocationTargetException e) {
		fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
	} catch (ClassCastException e) {
		fatalError("Set을 구현하지 않은 클래스입니다."); 
	}
	
	// 생성한 집합을 사용한다. 
	s.addAll(Arrays.asList(args).subList(1, args.length)); 
		System.out.println(s);
}
private static void fatalError(String msg) {
	System.err.println(msg);
	System.exit(1); 
}
```
**단점**
1. 런타임에 총 6개의 예외를 던질 수 있다.
	* 리플렉션 없이 생성했다면, 이 오류는 컴파일에 모두 잡을 수 있는 예외처리다.
	* java7부터 리플렉션 예외를 잡는 ReflectiveOperationException 을 호출할 수 있다.
2. 리플렉션이 아니였다면 한 줄로도 충분히 구현할 수 있는 기능이다.
3. 비검사 형변환 경고를 뜨게 한다.
	* (Item27) 이 경고를 숨기기 위해서는 개발자의 책임이 뒤따른다.

**장점**
* 런타임에 존재하지 않을 수 있는 다른 클래스, 메서드 필드와의 의존성을 관리할 때 적합하다.
	* 버전이 여러개 존재하는 외부 패키지를 다룰 때 유용하다.
	* 가장 오래된 버전만을 지원하도록 컴파일 후 -> 이후 버전의 클래스와 메서드를 리플렉션을 통해 접근한다.



<!--
```java

```
 -->