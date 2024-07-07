# Item 10 - equals는 일반 규약을 지켜 재정의하기

#### equals 메서드 재정의가 필요없는 상황
1. 각 인스턴스가 본질적으로 고유할 때
	* 값을 표현하는 것이 아닌 동작하는 개체를 표현하는 클래스
		ex) Thread
2. 인스턴스의 논리적 동치성을 검사할 일이 없을 때
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때
	ex) Set - AbstractSet, List - AbstractList ...
4. 클래스가 private거나 package-private이고 equals메서드를 호출할 일이 없을 때
	ex) equals 함수 호출 금지
	```java
	@Override public boolean equals(Object o) {
		throw new AssertionError(); // 호출 시 에러
	}
	```
5. 인스턴스 통제 클래스일 경우
	* 어차피 인스턴스가 2개 이상 만들어지지 않으니 필요가 없다.

#### equals 메서드 재정의가 필요한 상황
* 상위 클래스가 논리적 동치성을 비교하도록 재정의가 되지 않았을 때
* Map의 키, Set의 원소로 사용하기 위해

#### equals 메서드 재정의 시 일반 규약

1. 반사성 : x -> x
2. 대칭성 : x -> y & y -> x
	> ex) java.sql.Timestamp 는 java.util.Date를 확장한 후 nanosecounds 필드를 추가함.
	> => TimeStamp의 equals는 대칭성을 위배하고 있음.
	> * TimeStam의 API설명에는 Date 와 섞어 쓸 때의 주의사항 언급하고 있다. 
3. 추이성 : x -> y & y -> z & z -> x
4. 일관성 : 항상 같은 값이 나와야한다.
5. null 아님 : x -> null == false

```java
public class ColorPoint { 
	private final Point point; // 상속 대신 컴포지션 활용
	private final Color color;
	public ColorPoint(int x, int y, Color color) { 
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color); 
	}
// 이 ColorPoint의 Point 뷰를 반환한다.
	public Point asPoint() {
		return point; 
	}
	@Override public boolean equals(Object o) { 
		if (!(o instanceof ColorPoint))
			return false;
		ColorPoint cp = (ColorPoint) o;
		return cp.point.equals(point) && cp.color.equals(color);
	}
}
```

* float 과 double을 제외한 기본 타입 필드는 == 연산자로 비교한다.
* 참조타입 필드는 각각 equals 메서드로 비교한다.
* float 과 double 필드는 각각 정적 메서드인 Float.compare(f, f), Double.compare(d, d)로 비교한다.
	-> Float.Nan, -0.0f 와 같은 특수한 부동 소수 값을 다뤄야 하기 때문이다.

---

* equals를 재정의 할 때 hashCode도 재정의할것
* 매개변수를 반드시 Object로 둘 것. (@Override 사용하기)
