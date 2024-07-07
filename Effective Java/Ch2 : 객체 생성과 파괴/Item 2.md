# Item 2 - 생성자에 매개변수가 많다면 빌더를 고려하기

선택적 매개변수가 많을 시 대응 법으로는 3가지가 있다.  
1) 점층적 생성자 패턴
2) 자바빈즈 패턴
3) 빌더 패턴


#### 1. 점층적 생성자 패턴

```java
public class Alphabet { 
	private final int a; // 필수
	private final int b; // 필수
	private final int c; // 선택
	private final int d; // 선택
	private final int e; // 선택
	private final int f; // 선택
	public Alphabet(int a, int b) {
		this(a, b, 0); 
	}
	public Alphabet(int a, int b, int c) {
		this(a, b, c, 0); 
	}

	... 이하 생략
}
```

* 문제점
```java
  Alphabet alpha = new Alphabet(1, 10, ?, 12, ?, 21);  
```
1) 사용자가 설정하기 원하지 않는 매개변수까지 지정해줘야함.  
2) 매개변수가 많아지면 클라이언트 코드를 해석하기 어렵다.  
	ex) 타입이 가은 매개변수가 연달아 있으면 컴파일 시 에러를 잡을 수 없다.  

#### 2. 자바빈즈 패턴

```java
public class Alphabet {
	private final int a = -1; // 필수; 기본값 없음
	private final int b = -1; // 필수; 기본값 없음
	private final int c; // 선택
	private final int d; // 선택
	private final int e; // 선택
	private final int f; // 선택

	public Alphabet() {}

	public void setA(int val) { a = val; }
	public void setB(int val) { b = val; }
	public void setC(int val) { C = val; }
	
	... 이하 생략
}
```

* 문제점

```java
  Alphabet alpha = new Alphabet();
  alpha.setA(100);
  alpha.setB(201);
  alpha.setF(1);
```
1) 객체를 생성하기 까지 여러번의 메서드를 호출해야함.  
2) 객체가 완전히 생성되기 전까지는 `일관성`이 보장되지 않는다.  

=> 클래스를 불변으로 만들 수 없다.  
=> 쓰레드 안정성을 얻으려면 추가적인 작업 필요  
<!-- 추가적인 작업에 대해 구체적으로 -->

> **해결책**
> 
>   freeze() 메서드 호출을 통해 생성이 끝난 객체임을 보장한다.
> 	하지만, 컴파일 시 검증할 수 없다. -> 런타임 오류로 확인해야함.

#### 3. 빌더패턴

```java
public class Alphabet {
	private final int a; 
	private final int b;
	private final int c; 
	private final int d; 
	private final int e; 
	private final int f; 

	public static class Builder {
		// 필수 매개변수
		private final int a, b;

		// 선택 매개변수 - 기본값으로 초기화한다.
		private final int c = 0;
		private final int d = 0;
		private final int e = 0;
		private final int f = 0;

		public Builder(int a, int b) {
			this.a = a;
			this.b = b;
		}

		public Builder c(int val) { c = val; return this; } 
		public Builder d(int val) { d = val; return this; }
		public Builder e(int val) { e = val; return this; }
		public Builder f(int val) { f = val; return this; }
		// 자신을 반환하기 때문에 연속적으로 호출 가능 
		// 이런 형태를 플루언트 API 혹은 매서드 연쇄라고 한다.

		public Alphabet build() { return new Alphabet(this); }
	}
	
	private Alphabet(Builder builder) {
		// 인자 유효성 검사를 하려면 해당 위치에서 진행함.
		a = builder.a;
		b = builder.b;
		c = builder.c;
		d = builder.d;
		e = builder.e;
		f = builder.f;
	}
}
```

```java
// 클라이언트 사용 예시
  Alphabet alpha = new Alphabet.Builder(1, 10).f(102).c(200).build();
```
빌더 패턴은 계층적 패턴에서 사용하기 좋다. -> Builder클래스 상속  


- 빌드 패턴 단점
	1. 추가적인 빌더 생성 비용 발생
	2. 매개변수 4개 이상에서 효율 ( 하지만 API는 시간이 지남에 따라 매개변수가 많아짐)

















