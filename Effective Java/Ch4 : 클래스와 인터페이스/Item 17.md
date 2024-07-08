 # Item 17 - 변경 가능성을 최소화하라

 #### 불변 클래스란?
인스턴스의 내부 값을 수정할 수 없는 클래스이다.
ex ) String, 기본 타입의 박싱된 클래스들, BigInteger, BigDecimal

* 가변 클래스보다 설계, 구현, 사용이 더 쉽다.
* 오류 발생 여지도 적고 훨씬 안전하다.


#### 불변 클래스 생성 방법

1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
2. 클래스를 상속 불가능으로 만든다.
	* 클래스를 final 로 선언한다.
	* 모든 생성자를 private || package-private로 만들고 public 정적 팩터리 제공
3. 모든 필드를 final로 선언한다.
4. 모든 필드를 private으로 선언한다.
	* public final은 추후 API 리팩토링의 유연성을 저해할 수 있기 때문에 주의해야 한다.
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

```java
// 복소수 클래스
static final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                           re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                           (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Complex complex = (Complex) o;
        return Double.compare(complex.re, re) == 0 && Double.compare(complex.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return Objects.hash(re, im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
* 사칙연산 메서드들은 새로운 complex 인스턴스를 만들어 반환.
	* 사칙연산 메서드는 피연산자의 인스턴스는 그대로, 결과는 새 Complex 인스턴스를 만들어 반환한다.
	* 이를 함수형 프로그래밍이라고 한다. </br>
	* 절차적, 명령형 프로그래밍에서는 피연산자인 자신을 수정하여 상태가 변하게 된다.


#### 불변 클래스의 장점
1. Thread-safe 성격을 가져서 따로 동기화할 필요가 없다.
	* 자주 쓰이는 값들을 상수로 제공하기
2. 자주 사용되는 인스턴스를 캐싱 -> 정적 팩터리 생성 가능 (인스턴스 중복 생성 방지)
3. 방어적 복사가 필요없다. 
	* 볼변이라 유일하기 때문.
4. 불변 객체는 자유롭게 공유가 가능하며 불변 객체끼리 네부 데이터를 공유할 수 있다.
5. Map / Set의 구성요소로 사용하기
	* 원소의 값이 바뀌면 불변식이 허물어지는데, 불변 객체는 그렇지 않다.
6. `실패 원자성` 을 가진다.
	* 메서드에서 예외가 발생한 후에도 객체는 여전히 메서드 호출 전과 똑같은 유효한 상태다.	
	

#### 불변 클래스의 단점
1. 값이 다르면 반드시 독립된 객체로 만들어야한다.
	* 그 값이 1bit가 다르더라도 다시 만들어야한다.
	* 해결책
	클라이언트들이 원하는 다단계 연산을
	> 예측할 수 있다면,  기본 기능으로 제공하는 가변 동반 클래스를 제공한다. ex ) StringBuilder, StringBuffer
	> 예측할 수 없다면,  클래스를 public 으로 제공한다. ex ) String

<!-- 가변 동반 클래스란? -->
	> **가변 동반 클래스**란?
	> 본래 불변인 객체의 관련값을 수정해 줄 수 있는 클래스


#### 불변 클래스 설계 방법
자신을 상속하지 못하게 한다면, 클래스가 불면임이 보장된다.
1. final 클래스 선언
2. 모든 생성자를 private || package-private로 만들고 public 정적 팩터리 제공

```java
// 정적 팩터리를 사용한 불변 클래스
public class Complex {
  private final double re;
  private final double im;

  private Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }

  public static Complex valueOf(double re, double im) {
    return new Complex(re, im);
  }
}
```
* 다음 릴리즈에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수 있다.


3. 무조건 setter를 만들지 말자.
4. 합당한 이유가 없다면 모든 필드는 private final이여야 한다.
5. 생성자는 초기화가 완벽히 끝난 상태의 객체를 생성해야한다.