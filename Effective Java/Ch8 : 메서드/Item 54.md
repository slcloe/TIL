# Item 54 - null이 아닌, 빈 컬렉션이나 배열을 반환하기

#### null을 반환할 때의 문제점
```java
private final List<Cheese> cheesesInStock = ...;
/**
* @return 매장 안의 모든 치즈 목록을 반환한다. 
* 단, 재고가 하나도 없다면 null을 반환한다. 
*/
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null
		: new ArrayList<>(cheesesInStock); 
}
```
* null 을 반환한다면 클라이언트는 null을 처리하는 방어 코드를 추가로 작성해야한다.
```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
	System.out.println("null 가드");
```
* 널가드를 빼먹으면 오류가 생길 수 있다.
* null을 반환하는 쪽에서도 상황에 대해 케이스를 따로 분류해줘야하는 번거로움이 있다.

* 빈 컨테이너 보단 null반환이 나을까?
1. 빈 컨테이너르 생성하는데 소요되는 시간은 크게 신경 쓸 수준이 되지 못한다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고 정적객체로 생성하여 반환할 수 있다.
	* 매번 똑같은 빈 `불변`컬렉션을 반환한다.
	* Collections.emptyList(), Collections.emptySet(), Collections.emptyMap()
##### 빈 컬렉션 객체 반환하기
```java
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList()
		: new ArrayList<>(cheesesInStock);
}
```
##### 빈 배열 반환하기
```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(new Cheese[0]); 
}
```
* 성능이 우려된다면, 길이가 0인 배열은 모두 불변이기 때문에 미리 선언해둬도 상관없다.
* toArray()는 배열의 크기에 따라 동작이 달라진다.
	* 배열이 충분히 크면 원소를 담아 반환하고
	* 그렇지 않으면 배열을 새로 만들어 그 안에 원소를 담아 반환한다.


<!--
```java

```
 -->
