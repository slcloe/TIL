# Item 34 - int 상수 대신 열거 타입을 사용하기

#### 정수 열거 패턴

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
``` 
##### 단점
* 타입 안전이 보장되지 않는다.
* 동작 과정에 오류가 있더라도 런타임 시점에도 잡을 수 없다.
	* 정수 열거 패턴을 위한 namespace를 지원하지 않기 때문
* 상수의 값이 바뀌면 클라이언트도 반드시 컴파일 해야한다. (하지만 컴파일하지 않은 쪽에서는 실행이 되도 엉뚱하게 동작하고 에러를 뱉어내지 않는다.)
* 문제열로 출력할 시 규칙이 애매하다.

#### 문자열 열거 패턴
* 이건 정수 상수 패턴보다 더 나쁘다.
* 오타를 컴파일러가 확인할 수 없고 런타임 오류를 유도한다.
* 문자열 비교에 따른 성능 저하가 발생한다.


#### 열거 타입
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum ORANGE { NAVEL, TEMPLE, BLOOD }
``` 
* 완전한 형태의 클래스이다.
* 상수 하나당 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
	* 싱글톤이다.
* 컴파일 시 타입 안정성을 제공한다.
	* 다른 열거 타입 끼리 == 연산자로 비교하면 컴파일 에러를 도출한다.
* namespace 제공
	* 이름이 같은 상수도 공존할 수 있다.
		ex) A.NUMBER B.NUMBER
* toString()으로 출력할 ㅜㅅ 있다.
* 메서드, 필드를 추가할 수 있고 Comparable, Serializable과 같은 인터페이스도 구현 가능하다.

#### ENUM 사용 예시
```java
enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass; // 질량 (단위: 킬로그램)
    private final double radius; // 반지름 (단위: 미터)
    private final double surfaceGravity; // 표면중력 (단위: m / s^2)

    // 중력상수 (단위: m^3 / kg s^2)
    private static final double G = 6.677300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
``` 
* 특정 필드에 값을 할당하려면 생성자를 통해 할당하면 된다.
* 열거 타입도 되도록 private, package-private 메서드로 구현하자

</br>

#### ENUM.values()

```java
public class WeightTable {
	public static void main(String[] args) {
		double earthWeight = Double.parseDouble(args[0]);
		double mass = earthWeight / Planet.EARTH.surfaceGravity(); 
		for (Planet p : Planet.values())
			System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));
	} 
}
``` 
* 자신 안에 정의도니 상수들의 값을 배열에 담아 반환하는 정적 메서드 values()를 제공
	선언된 순서대로 저장된다.
* 혹여 열거 타입을 하나 삭제한다 하더라도, values()의 size-- 되고 나머지 값은 동일하다.

#### 상수별 메서드 구현 : switch

```java
public enum Operation {
	PLUS, MINUS, TIMES, DIVIDE;
	// 상수가 뜻하는 연산을 수행한다.
	public double apply(double x, double y) {
		switch(this) {
			case PLUS: return x + y; 
			case MINUS: return x - y; 
			case TIMES: return x * y; 
			case DIVIDE: return x / y;
		}
		throw new AssertionError("알 수 없는 연산: " + this); 
	}
}
``` 
**단점**
* 새로운 상수를 추가할 때마다 case를 추가해야한다.
* 실제 도달할 일이 없는 throw문을 기술적인 문제로 써야만 한다.


#### 상수별 메서드 구현 : 추상 메서드
```java
public enum Operation {
	PLUS {public double apply(double x, double y){return x + y;}}, 
	MINUS {public double apply(double x, double y){return x - y;}}, 
	TIMES {public double apply(double x, double y){return x * y;}}, 
	DIVIDE{public double apply(double x, double y){return x / y;}};

	public abstract double apply(double x, double y); 
}
``` 
* 새로운 상수를 추가할 때, apply함수를 재정의하지 않았다면 컴파일 오류로 알려준다.


#### 상수별 메서드 구현 : 상수별 메서드 구현 + 상수별 데이터
```java
public enum Operation { 
	PLUS("+") {
	public double apply(double x, double y) { return x + y; } },
	MINUS("-") {
	public double apply(double x, double y) { return x - y; } },
	TIMES("*") {
	public double apply(double x, double y) { return x * y; } },
	DIVIDE("/") {
	public double apply(double x, double y) { return x / y; } };

	private final String symbol;

	Operation(String symbol) { this.symbol = symbol; }

	@Override public String toString() { return symbol; }
	public abstract double apply(double x, double y); 
}

public static void main(String[] args) { 
	double x = Double.parseDouble(args[0]); double y = Double.parseDouble(args[1]); for (Operation op : Operation.values())
	System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
``` 
* symbol 추가
* toString() 재정의를 함으로써 계산식 출력이 더 편리해졌다.

#### 열거타입의 fromString()
```java
private static final Map<String, Operation> stringToEnum = 
	Stream.of(values()).collect(toMap(Object::toString, e -> e));
// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
	return Optional.ofNullable(stringToEnum.get(symbol)); 
}
```
* 열거 타입 상수 생성 후 정적 필드가 초기화 될 때 stringToEnum 맵에 추가된다.
* 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다.
	* 만약 코드가 컴파일 되었다 하더라도 런타임 시 NullPointerException이 발생한다.
	* 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수 뿐이다.

#### 열거 타입 상수끼리 코드를 공유하기 어렵다 : switch
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;

        switch(this) {
            // 주말
            case SATURDAY : case SUNDAY :
                overtimePay = basePay / 2;
                break;
            // 주중
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
}
``` 
* 새로운 값을 열거 타입에 추가하려면 case문에도 추가해줘야한다.
**해결 방법**
1. 모든 상수에 계산코드를 추가한다.
2. 평일용과 주말용으로 계산되는 코드를 도우미 메서드로 작성한다.
=> 가독성도 안좋고, 오류 발생 가능성이 높아진다.
* 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 좋다.


#### 열거 타입 상수끼리 코드를 공유하기 어렵다 : 전략 상수 패턴

```java
enum PayrollDay {
    MONDAY(PayType.WEEKDAY)
    , TUESDAY(PayType.WEEKDAY)
    , WEDNESDAY(PayType.WEEKDAY)
    , THURSDAY(PayType.WEEKDAY)
    , FRIDAY(PayType.WEEKDAY)
    , SATURDAY(PayType.WEEKEND)
    , SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    public int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            @Override
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            @Override
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
``` 
* 새로운 상수를 추가할 때 생성자에서 PayType을 선택하도록 위임한다.
* 상수별 메서드 구현이 필요없다.
* 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하기


#### 열거 타입을 사용해아 할 때
* 필요한 원소를 컴파일타임에 모두 알 수 있는 상수의 집합일 때
	ex. 연산 코드, 명령줄 플래그
* 열거 타입에 정의된 상수 개수가 영원히 고정, 불변일 필요는 없다.


<!-- 
```java

``` 
5:10
-->
