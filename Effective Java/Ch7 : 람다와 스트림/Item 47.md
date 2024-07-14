# Item 47 - 반환 타입으로는 스트림보다 컬렉션을 택하기


#### 원소 시퀀스
* 자바8 이전에는 Iterable, Collection 을 사용해서 원소를 반환했다.
* 자바8 이후에 stream을 사용해 원소를 반환할 수 있지만,
* 스트림은 Iterable 을 extends하지 않기 때문에 Iterator을 지원하지 않는다.

```java
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
// 프로세스를 처리한다. }
``` 
* iterator를 지원하지 않기 때문에 타입 추론이 불가능하고 위 코드는 컴파일 오류가 난다.
* 이를 해결하기 위해서 형변환을 거치지만,
```java
for (ProcessHandle ph : (Iterable<ProcessHandle>)
ProcessHandle.allProcesses()::iterator){
 // 프로세스를 처리한다. }
``` 
* 위 코드는 에러는 나지 않지만 난잡하고 직관성이 떨어진다.


#### 어뎁터 메서드

```java
// stream To iterator
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
return stream::iterator; }

// iterator To stream
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
return StreamSupport.stream(iterable.spliterator(), false); }
``` 
* 어뎁터 메서드로 타입을 변환하는 메서드를 구현했다.
* Collection 인터페이스는 stream, iterable 을 모두 구현하기 때문에 이렇게 구현할 바에는 Collection이나 하위 타입을 쓰는것이 최선이다.

#### Collection 활용 : 멱집합
**멱집합**이란?
	* 한 집합의 모든 부분집합을 원소로 하는 집합 (공집합 제외)

```java
public class PowerSet {
public static final <E> Collection<Set<E>> of(Set<E> s) {
List<E> src = new ArrayList<>(s); if (src.size() > 30)
throw new IllegalArgumentException(
"집합에 원소가 너무 많습니다(최대 30개).: " + s);
return new AbstractList<Set<E>>() { @Override public int size() {
// 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
return 1 << src.size(); }
@Override public boolean contains(Object o) {
return o instanceof Set && src.containsAll((Set)o); }
@Override public Set<E> get(int index) {
Set<E> result = new HashSet<>();
for (int i = 0; index != 0; i++, index >>= 1)
if ((index & 1) == 1) result.add(src.get(i));
return result; }
}; }
}
``` 
* AbstractCollection은 Iterable 함수 외에 contains(), size() 만 구현하면 된다. 
	-> Collection을 반환하기 좋은 예시다.
* Stream은 size()를 구할 수 없다.

#### Stream 활용 : 부분 리스트

```java
public class SubLists {
public static <E> Stream<List<E>> of(List<E> list) {
return Stream.concat(Stream.of(Collections.emptyList()), prefixes(list).flatMap(SubLists::suffixes));
}
private static <E> Stream<List<E>> prefixes(List<E> list) { return IntStream.rangeClosed(1, list.size())
.mapToObj(end -> list.subList(0, end)); }
private static <E> Stream<List<E>> suffixes(List<E> list) { return IntStream.range(0, list.size())
.mapToObj(start -> list.subList(start, list.size())); }
}
```
* 반복이 더 자연스러운 상황에도 스트림을 사용할 수 있다.
* 이럴 때는 위 설명에 있던 어댑터 함수를 사용하자!
	* 하지만 코드가 더러워지고, 성능상 불이익을 가져온다.
</br>

* 원소 시퀀스를 반환하는 메서드를 작성할 때, stream과 iterator 처리 모두 구현하자
* 기본적으로는 Collection에 담아 반환하는 것이 유연하다.






```java

``` 

```java

``` 

```java

``` 

<!-- 
```java

``` 
-->