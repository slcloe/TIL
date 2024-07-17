# Item 55 - Optional 반환은 신중하기

#### Optional<T>
* null이 아닌 T타입 참조를 하나를 담는다.
* 원소를 최대 1개를 가질 수 있는 불변 컬렉션이다.
* 옵셔널을 반환하면 예외를 던지는 메서드 보다 유연하고 사용하기 쉽다.
* null을 반환하는 메서드보다 오류 가능성이 적다.

#### Optional 사용 예시
* 컬렉션에서 최대값을 구하는 예시
* 예외 : 컬렉션이 비어있는 경우 
##### throw exception
```java
public static <E extends Comparable<E>> E max(Collection<E> c) { 
	if (c.isEmpty())
		throw new IllegalArgumentException("빈 컬렉션");
	E result = null; 
	for (E e : c)
		if (result == null || e.compareTo(result) > 0) 
			result = Objects.requireNonNull(e);
	return result; 
}
```
* null 반환 시 IllegalArgumentException을 던진다.

##### return optional 
```java
public static <E extends Comparable<E>> 
				Optional<E> max(Collection<E> c) {
	if (c.isEmpty())
		return Optional.empty();
	E result = null; 
	for (E e : c)
		if (result == null || e.compareTo(result) > 0) 
			result = Objects.requireNonNull(e);
	return Optional.of(result); 	
}
```
* 컬렉션이 비었다면, return Optional.empty()
* 컬렉션이 비어있지 않다면, return Optional.of(result)
* **Optional을 반환하는 메서드에서 null을 반환하지 말자!**

##### optional 과 stream
```java
public static <E extends Comparable<E>>
	Optional<E> max(Collection<E> c) {
	return c.stream().max(Comparator.naturalOrder()); 
}
```
* 스트림의 종단 연산 중 상당수가 optional 을 반환한다.
> Optional은 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.
> 예외를 처리하는 검사 예외와 비슷하지만, 검사 예외를 던지면 클라이언트에서 반드시 예외 처리 코드를 넣어야한다.

#### Optional 활용

##### default 값 정하기
**optional.orElse()**
```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```
* Optional의 값에 null이 주어진다면, default값을 반환한다.

##### 원하는 예외 던지기
**optional.orElseThrow()**
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```
* 예외가 실제로 발생하지 않는한 예외 생성 비용은 들지 않는다.

##### 항상 값이 채워져있다고 가정하기
**optional.get()**
```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```
* 항상 값이 채워져 있다고 확신할때 쓰는 함수이다.
* 하지만 null이 들어갔을 경우, NoSuchElementException이 발생한다.

##### 초기 생성 비용을 낮출 때
**optional.orElseGet()**
```java
Connection connection = getConnection(dataSource)
							.orElseGet(() -> getLocalConnection());
```
* 값이 처음 필요할 때 `Supplier<T>`를 생성하기 때문에 초기 생성 비용을 낮출 수 있다.

##### 안전 처리를 위해
**optional.isPresent**

```java
Optional<ProcessHandle> parentProcess = ph.parent(); 
System.out.println("부모 PID: " + 
		(parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));

System.out.println("부모 PID: " + 
		ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```
* 부모 프로세스의 프로세스 ID를 출력하고, 만약 부모가 없다면 "N/A"를 출력하는 코드이다.

```java
//java8
streamOfOptionals.filter(Optional::isPresent) 
				 .map(Optional::get)

//java9
streamOfOptionals.flatMap(Optional::stream)
```
* stream을 사용할 때 옵셔널들을 `Stream<Optional<T>>`로 받아 처리할 수 있다.
* java9 에서는 Optional 을 stream 으로 바꿔주는 메서드가 생겼다.
	* Optional to Stream 의 어댑터 역할을 한다.

#### Optional 구현 시 주의 사항

* Optional로 반환 값을 사용하면 안되는 경우
	* Collection, Strea, 배열, Optional 같은 컨테이너 타입을 반환값으로 두는 경우
		* 이럴 경우는 빈 `List<T>`를 반환하는게 좋다.
		* 빈 컨테이너를 그대로 반환하면 클라이언트에서 옵셔널 처리 코드를 넣지 않아도 되기 때문 (Item 54)
	* 성능이 중요할 때
		* Optional도 새로 할당하고 초기화해야하는 객체이기 때문에 성능 이슈가 있을 수 있다. (객체를 사용하기 위해서 get함수에 한번 더 접근해야함.)
		* 이럴땐, null을 반환하거나 예외를 더니는 편이 나을 수 있다.
		* 특히, `박싱된 기본 타입`을 담은 옵셔널을 반환하지 말자
			* OptionalInt와 같은 전용 클래스를 사용하자

	* 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하지 말자
		* 복잡성이 높아 혼란, 오류를 야기한다.

* Opitonal로 반환 값을 사용해야 하는 경우
	* 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 할때
		

<!--
```java

```
 -->
