# Item 38 - 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하기

#### 열거 타입의 확장
* 열거 타입은 확장을 할 수 없다.
	* 확장된 타입의 순회를 할 방법이 없다.
	* 확장성을 높이려면 고려할 요소가 많기 때문에 설계/구현이 복잡해진다.
=> 열거 타입의 확장을 인터페이스를 통해 구현하기!

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x+y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x-y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x*y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x/y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}

public enum ExtendedOperation implements Operation {
    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
``` 
* BasicOperation 은 확장할 수 없지만, Operation은 확장 가능하다.

##### class 리터럴로 실행시키기
```java
public static void main(String[] args) { 
	double x = Double.parseDouble(args[0]); 
	double y = Double.parseDouble(args[1]); 
	test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test( Class<T> opEnumType, double x, double y) {
	for (Operation op : opEnumType.getEnumConstants()) 
		System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
``` 
* class리터럴을 한정적 타입 토큰을 사용하여 타입을 제한했다.
* `<T extends Enum<T> & Operation>`는 class객체가 열거 타입인 동시에 Operation의 하위 타입이라는 뜻이다.

#### 컬렉션으로 실행시키기
```java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]); 
	test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet, double x, double y) {
	for (Operation op : opSet) 
		System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}

``` 
* 한정적 와일드 카드 타입을 넘겨 실행한 예시이다.
* 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다는 단점이 있다.

#### 인터페이스를 통한 열거 타입 확장법의 단점
* 열거 타입끼리는 구현을 상속할 수 없다.
	1. 인터페이스의 디폴트 메서드를 활용하자!
	2. 도우미 클래스다 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없애자

<!-- 
```java

``` 
-->