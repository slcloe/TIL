# Item 29 - 이왕이면 제네릭 타입으로 만들기

##### 제네릭으로 없이 구현하기
```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}

		Object result = elements[size];
		elements[size] = null;

		return result;
	}

	private void ensureCapacity() {
		if(elements.length == size)
			elements = Arrays.copyOf(elements, 2 * size + 1);
	}
}
``` 
* 제네릭 타입을 사용하지 않고 단순히 Object를 사용해 구현한 스택 코드


##### 제네릭으로 구현하기
* 클래스 선언에 타입 매개변수를 추가한다.
```java
private E[] elements;
private int size = 0;
private static final int DEFAULT_INITIAL_CAPACITY = 16;

public Stack() {
	elements = new E[DEFAULT_INITIAL_CAPACITY];
}
``` 
* 제네릭은 E와 같이 실체화 불가 타입으로는 배열을 만들 수 없다.
* 위 코드에서는 generic array creation 에러가 발생한다.

#### 제네릭 타입 배열 생성 방법 2가지
##### 1. Object배열 생성 후 E()로 형변환하기
```java
// 배열 elements는 push(E)로 넘어온 E인스턴스만 담는다.
// 타입 안정성 보장함!
// 이 배열의 런타임 타입은 Object[]이다.
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
``` 
* 컴파일 시 타입 변환이 안전한지 알수 없기 때문에 경고를 띄운다.
* 하지만 이 매서드는 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 없기 때문에 배열에 저장되는 원소의 타입은 항상 E라고 할 수 있다.
 -> @SupressWarnings를 통해 비검사 경고를 지워도 된다.
**특징**
* 구현이 간단하다. 
	배열 생성 시 형변환을 한번만 수행해도 된다.
* 가독성이 더 좋고, 코드도 짧다.

##### 2. 변수를 형변환하기
```java
public E pop() {
  if(size == 0) {
    throw new EmptyStackException();
  }
  @SuppressWarnings("unchecked") E result = (E) elements[--size];

  elements[size] = null; // 다 쓴 참조 해제
  return result;
}
``` 
* push 에서 E 타입만 허용하므로 형변환은 안전하다.
* -> SupressWarnings를 통해 비검사 경고를 지워도 된다.

**특징**
* 구현 시, 배열에서 원소를 읽을 때마다 해줘야한다.
* 힙오염이 생기지 않는다.

>
> 실무에서는 1번째 방법을 더 선호한다.
> 하지만 배열의 타입이 컴파일과 런타임시점에서 같지 않기때문에 힙오염을 일으킨다.
>

<!-- 
```java


``` 
-->