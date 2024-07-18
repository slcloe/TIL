# Item 58 - 전통적인 for 문보다는 for-each문을 사용하기

#### for 문
```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) { 
	Element e = i.next();
	... // e로 무언가를 한다. 
}
```

```java
for (int i = 0; i < a.length; i++) {
	... // a[i]로 무언가를 한다. 
}
```
* while 보다는 낫지만, 변수를 잘못 사용할 수 있는 위험이 있다.

#### for-each 문
```java
for (Element e : elements) {
	... // e로 무언가를 한다. 
}
```

```java
for (Suit suit : suits)
	for (Rank rank : ranks)
		deck.add(new Card(suit, rank));
```
* 간단하게 반복문을 구성하며 코드도 더 깔끔해진다.
* 반복 대상에 구애받지 않고, 순회 속도는 그대로고, 최적화된 속도를 따른다.
* 중첩 순회일 경우, index를 직접 건드리지 않기 때문에 실수할 일도 적다.


#### for-each 를 사용할 수 없는 경우
* 파괴적인 필터링
	* 순회하면서 선택된 원소를 지워가는 필터링이다.
	* java8 부터 Collection 의 removeIf메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

* 변형
	* 리스트 혹은 배열을 순회하면서 해당 원소의 값을 수정할 수 없다.

* 병렬 반복
	* 여러 컬랙션을 병렬로 순회해야한다면, 반복자와 인덱스 변수를 사용해서 명시적으로 제어해야한다.

#### Iterable
```java
public interface Iterable<E> {
	// 이 객체의 원소들을 순회하는 반복자를 반환한다.
	Iterator<E> iterator(); 
}
```
* for-each를 사용하기 위해서는 반드시 구현해야함.
<!--

 -->