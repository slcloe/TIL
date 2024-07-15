# Item 52 - Overloading는 신중히 사용하기


#### Overloading의 문제점
```java
public class CollectionClassifier {
public static String classify(Set<?> s) {
return "집합"; }
public static String classify(List<?> lst) {
return "리스트"; }
public static String classify(Collection<?> c) {
return "그 외"; }
public static void main(String[] args) { Collection<?>[] collections = {
new HashSet<String>(),
new ArrayList<BigInteger>(),
new HashMap<String, String>().values()
};
for (Collection<?> c : collections) System.out.println(classify(c));
} }
```
* 예상결과 : 집합 리스트 그 외
* 실행결과 : 그 외 그 외 그 외
* 오버로딩은 어느 메서드를 호출할지가 컴파일 타임에 정해지기 때문이다.
	* 컴파일 시에는 for문 안에 있는 c는 항상 Collection 이다. -> 3번째 메서드만 호출됨.
* Override메서드는 동적으로 선택, Overload 메서드는 정적으로 선택

#### Overriding일 때,
```java
class Wine {
	String name() { return "포도주"; } 
}
class SparklingWine extends Wine {
	@Override String name() { return "발포성 포도주"; } 
}
class Champagne extends SparklingWine {
	@Override String name() { return "샴페인"; } 
}
public class Overriding {
	public static void main(String[] args) {
		List<Wine> wineList = List.of( new Wine(), new SparklingWine(), new Champagne());
		for (Wine wine : wineList) 
			System.out.println(wine.name());
	}
}
```
* Override는 가장 하위에서 정의한 메서드가 실행된다.
* 런타임 시 결정된다.

#### Overloading 을 쓰지 않고 해결하기
```java
public static String classify(Collection<?> c) { 
	return c instanceof Set ? "집합" : 
		   c instanceof List ? "리스트" : "그 외";
}
```
* classify 메서드를 하나로 합치고 instanceof 로 명시적 검사로 해결했다.

#### Oveloading 구현법
* 오버로딩이 혼동되는 (위에 예시와 같은) 상황을 피하기
* 매개변수 수가 같은 오버로딩은 만들지 말기
	* 가변인수를 사용하는 메서드라면 오버로딩은 아예 하지말아야한다.
* 오버로딩 대신 메서드의 이름을 다르게 지어주기
	* ObjectOutputStream 은 writeBoolean, writeInt <-> readBoolean, readInt로 오버로딩을 피했다.
* 생성자의 경우, 정적 팩터리를 사용하자.
* 오버로딩 시 서로 형변환을 할 수 있는 타입으로 메서드를 구성하자.
	* 매개변수들의 런타입 타입만으로 결정되니, 혼동될일이 없다.!

#### Overloading 주의점
1. 오토박싱
```java
public class SetList {
	public static void main(String[] args) {
		Set<Integer> set = new TreeSet<>(); 
		List<Integer> list = new ArrayList<>();
		for (int i = -3; i < 3; i++) { set.add(i);
			list.add(i);
		}
		for (int i = 0; i < 3; i++) { 
			set.remove(i);
			list.remove(i);
		}
		System.out.println(set + " " + list); 
	}
}
```
* 예상결과 : "[-3, -2, -1] [-3, -2, -1]"
* 실행결과 : "[-3, -2, -1] [-2, 0, 2]"
* list 측의 실행결과가 다른 이유는 remove(Object element) 와 remove(int index)가 혼동되어 사용되었기 때문이다.
	* 따라서 list.remove((Integer) i) 혹은 list.(Integer.valueOf(i)) 를 사용하면 문제를 해결할 수 있다.!
* set은 remove(Object element) 만 존재하기 때문에 결과가 동일하게 나왔다.

2. 람다와 메서드 참조
```java
// 1번. Thread의 생성자 호출
new Thread(System.out::println).start();

// 2번. ExecutorService의 submit 메서드 호출 
ExecutorService exec = Executors.newCachedThreadPool(); 
exec.submit(System.out::println);
```
* 결과적으로, 1번은 정상실행되지만 2번은 컴파일 오류가 난다.
* 1번은 Runnable을 구현한 Thread를 생성하고 있다.
* 2번은 exec.submit() 에서 Runnable을 구현하고 있지만, `submit(Callable<T>)` 메서드도 오버로딩 되어있다.
* 참조된 메서드(println)와 호출한메서드(submit)이 양쪽 다 오버로딩 되어 오버로딩 해소 알고리즘이 잘 동작하지 않았다.
* 즉, 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.
	* 함수형 인터페이스들은 근본적으로 다르다라고 취급하지 않는다.
	* -Xlint:overloads를 지정하면 이런 종류의 오버로딩을 경고해준다.

<!--
```java

```
 -->
