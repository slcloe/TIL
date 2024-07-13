# Item 35 - ordinal 메서드 대신 인스턴스 필드를 사용하기

#### ordinal이란?
: 열거 타입에서 몇 번째 위치인지 반환하는 메서드
* EnumSet, EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다.
* Enum API 문서에 따르면 대부분의 프로그래머는 이 메서드를 쓸 일이 없다고 적혀 있다.

#### ordinal을 잘못 사용한 예
```java
enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}
``` 
**문제점**
1. 상수의 선언 순서를 바꾸면 numberOfMusicians가 오작동한다.
2. 값을 중간에 비워둘 수 없다.

**해결책**
열거 타입 상수에 연결된 값은 각 인스턴스 필드에 저장하자


```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), NONET(9), DECTET(10), TRIPLE_QUARTET(12);
	private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size; }
	public int numberOfMusicians() { return numberOfMusicians; }
}
``` 

<!-- 
```java

``` 
-->