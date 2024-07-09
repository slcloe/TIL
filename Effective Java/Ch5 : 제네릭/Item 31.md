# Item 31 - 한정적 와일드카드를 사용해 API 유연성 높이기 

#### 와일드 카드 타입을 사용해야 하는 이유
* 제네릭은 불공변이다.
	-> `List<String>`은 `List<Object>`가 하는 일을 제대로 수행할 수 없다. / 하위타입이 될 수 없다.

```java
public class Stack<E> { 
	public Stack();
	public void push(E e); 
	public E pop();
	public boolean isEmpty();
}

public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}

public void popAll(Collection<E> dst) {
    while(size > 0) {
        dst.add(pop());
    }
}
```
```java
Stack<Number> stack = new Stack<>();
List<Integer> integers = List.of(1, 2, 3);
stack.pushAll(integers);
```
* Integer 은  Number의 하위 타입이지만, 제네릭은 불공변이기 때문에 에러가 발생한다.

#### Product : 한정적 와일드 카드 사용
```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```
* 위 코드는 에러가 발생하지 않는다.
* E를 상속한 타입이라면 어떤 타입이여도 괜찮다.

#### Consumer : 한정적 와일드 카드 사용
```java
public void popAll(Collection<? super E> dst) {
    while(size > 0) {
        dst.add(pop());
    }
}
```
* 상위 클래스로만 확장해야한다.

>  **Product vs Consumer**
>  * Produc 관점
>	 Stack.pushAll() 의 src 매개변수는 Stack이 사용할 E인스턴스를 생산한다
>	 -> <? extends E> 사용
>  * Consumer 관점
> 	 Stack.popAll()의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비한다.
>	 -> <? super E> 사용


##### super을 사용하지 않은 comparable예시
```java
public static <E extends Comparable<E>> E max(List<E> list);
```
* 위 코드에서 Comparable을 통해 max 를 구현한다고 한다.
* A와 A 를extends한 B가 왔을 때 어떤 클래스 기준으로 비교연산을 해야할지 모호하다.

```java
public static <E extends Comparable<? super E>> E max(List<E> list);
```
* 위와 같이 코드를 수정하면, 상위 객체 기준으로 Comparable을 동작할 수 있다.

#### 메서드 선언에 타입 매개변수가 한번만 나온다면 와일드 카드를 사용하자
```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
* `List<?>` 는 null이외에는 어떤 값도 넣을 수 없다.
* 따라서 private 메서드를 통해 와일드카드 타입을 실제 타입으로 바꿔주는 도우미 메서드를 구현하자.
* API외부로는 ? 와일드 카드가 보이고, 내부에서는 `<E>`타입을 쓰게 된다.
* -> 결과적으로 클라이언트는 복잡한swapHelper의 존재를 모른 채 그 혜택을 누리게 된다.
```
<!-- 
```java

```
 -->