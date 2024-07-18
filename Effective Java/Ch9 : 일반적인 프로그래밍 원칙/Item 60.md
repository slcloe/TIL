# Item 60 - 정확한 답이 필요하다면 float 와 double 은 피하기

#### float/double 이진 부동소수점의 취약점
```java
System.out.println(1.03 - 0.42);

// 실행결과 0.610000000000000
```
* float / double 타입은 과학과 공학 계산용으로 설계되었다.
	* 넓은 범위의 수를 빠르게 정밀한 근사치로 계산하기 위함이다.
	* 확실한 계산을 요구하는 작업과는 어울리지 않다. (금융 관련 계산)


#### 금융 계산 예시

```java
public static void main(String[] args) {
	double funds = 1.00;
	int itemsBought = 0;
	for (double price = 0.10; funds >= price; price += 0.10) {
		funds -= price;
		itemsBought++;
	}
	System.out.println(itemsBought + "개 구입");
	System.out.println("잔돈(달러):" + funds); 
}
```
* 사탕 3개를 구입한 후 잔돈은 0.3999999.. 달러가 출력되는 문제가 생긴다.
* **금융계산**에는 BigDecimal, int, long 타입을 사용하자!

##### BigDecimal 사용 예시
```java
public static void main(String[] args) {
	final BigDecimal TEN_CENTS = new BigDecimal(".10");
	int itemsBought = 0;
	BigDecimal funds = new BigDecimal("1.00"); 
	for (BigDecimal price = TEN_CENTS;
			funds.compareTo(price) >= 0;
			price = price.add(TEN_CENTS)) { 
		funds = funds.subtract(price); itemsBought++;
	}
	System.out.println(itemsBought + "개 구입"); 
	System.out.println("잔돈(달러): " + funds);
}

```
**단점**
* 기본타입보다 훨씬 불편하고 느리다.
* `BigDecimal(String)`을 사용한 이유는 계산 시 부정확한 값이 사용되는 것을 막기 위함이다. 

##### 정수타입 사용
```java
public static void main(String[] args) {
	int itemsBought = 0;
	int funds = 100;
	for (int price = 10; funds >= price; price += 10) {
		funds -= price;
		itemsBought++; 
	}
	System.out.println(itemsBought + "개 구입");
	System.out.println("잔돈(센트): " + funds); 
}
```
**단점**
* 크기가 제한된다.
* 소수점을 직접 관리해야한다.
<!--
```java

```
 -->