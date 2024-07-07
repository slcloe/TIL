# Item 2 - 생성자에 매개변수가 많다면 빌더를 고려하기
---

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
  Alphabet alpha = new (1, 10, ?, 12, ?, 21);  
```
1) 사용자가 설정하기 원하지 않는 매개변수까지 지정해줘야함.  
2) 매개변수가 많아지면 클라이언트 코드를 해석하기 어렵다.  
	ex) 타입이 가은 매개변수가 연달아 있으면 컴파일 시 에러를 잡을 수 없다.  

#### 2. 자바빈즈 패턴






