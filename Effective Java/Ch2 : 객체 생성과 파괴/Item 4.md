# Item 4 - 인스턴스화를 막기위해 private 생성자를 사용하기
---

##### 인스턴스화를 막아야할 때,
java.util.Collections 처럼 정적 멤버, 정적 메서드만 담은 유틸리티 클래스는 인스턴스 생성을 막아야한다.

`+ java8 부터 정적 메서드를 인터페이스에 넣을 수 있다. `


##### 1. 추상 클래스로 만들기
하위 클래스를 인스턴스화하면 된다. -> 오히려 상속해서 쓰라는 뜻이라고 오해할 수 도 있음.

##### 2. private 생성자를 만들기

```java
public class UtilityClass {
	private UtilityClass() {
		throw new AssertionError();
	}
}
```
* 상속도 불가능하게 한다.
	상위 클래스의 생성자를 호출할 때 private로 선언해 생성자의 접근이 불가하다.


	