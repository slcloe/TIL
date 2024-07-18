# Item 64 - 객체는 인터페이스를 사용해 참조하기

#### 인터페이스를 사용하여 참조하자
* 매개변수, 반환값, 변수, 필드 모두 인터페이스로 선언하자
* 객체의 실제 클래스를 사용해야 할 상황은 오직 생성자일 때 뿐이다.
* 인터페이스 타입을 사용하면 프로그래밍 유연해진다.
```java

Set<Son> sonSet = new LinkedHashSet<>();

LinkedHashSet<Son> sonSet = new LinkedHashSet<>();

```
* LinkedHashSet으로 선언하면 LinkedHashSet에 구현이 국한되지만,
* Set으로 선언하면 HashSet, TreeSet 등 Set을 구현한 다양한 클래스를 할당할 수 있다.

#### 하지만, 적합한 인터페이스가 없는 경우 클래스를 사용해야한다.
1. String, BigInteger 같은 값 클래스
	* 그래서 final 인 경우가 많다.
	* 값 클래스는 매개변수, 변수, 필드, 반환 타입으로 사용해도 무방하다.
2. 클래스로 작성된 프레임워크가 제공하는 객체
	* 추상클래스를 사용해 참조하자
	* ex. OutputStream
3. 인터페이스에는 없는 특별한 메서들르 제공하는 클래스
	* ex. PriorityQueue 는 Queue에 없는 compare메서드를 제공한다.


<!--
```java

```
 -->