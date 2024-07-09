# Item 22 - 인터페이스는 타입을 정의하는 용도로만 사용하기

#### 안티패턴 - 상수 인터페이스 
* 메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스를 뜻한다.

```java
상수 인터페이스
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 볼츠만 상수 (J/K)
    double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량 (kg)
    double ELECTRON_MASS = 9.109_383_56e-31;
}
```
* 클래스 내부에서 사용하는 상수는 내부 구현에 해당된다.
* 상수인터페이스 구현은 내부 구현을 클래스의 API로 노출하는 행위이다.
* 즉, 추후에 상수를 쓰지 않아도 여전히 해당 인터페이스를 구현하고 있어야한다.
ex. java.io.ObjectStreamConstants

#### 상수 유틸리티 클래스

```java
public interface PhysicalConstants {
	private PhysicalConstants() {} // 인스턴스화 방지

    // 아보가드로 수 (1/몰)
    double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 볼츠만 상수 (J/K)
    double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량 (kg)
    double ELECTRON_MASS = 9.109_383_56e-31;
}

```
* 내부 구현을 위한 상수는 클래스, 인터페이스 자체에 추가해야한다.
	ex. Integer.MAX_VALUE
* 열거타입으로 나타내기
* 위 두가지 방법이 불가하다면 유틸리티 클래스로 공개하기



