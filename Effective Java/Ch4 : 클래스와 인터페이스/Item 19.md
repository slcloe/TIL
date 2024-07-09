# Item 19 - 상속을 고려해 설계하고 문서화하고 그러지 않았다면 상속을 금지하기

#### 재정의 가능한 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야한다.

1. 어떤 순서로 호출하는지, 각 호출 결과가 이어지는 처리에 어떤 영향을 주는지
	* ex ) Hashmap.addAll()이 Hashmap.add()함수를 호출하여 구현되어있다. (Item18)
2. 재정의 가능 메서드 : public, protected 중 final이 아닌 메서드
3. 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남기기
	* ex )  백그라운드 스레드나 정적 초기화 과정에서도 호출이 일어날 수 있다.



#### Implementation Requirements와 @impleSpec 태그 사용
> Implementation Requirements : 해당 메서드의 내부 동작 방식을 설명하는 곳
> 메서드 주석에 @implSpec 태그를 붙여주면 자바독 도구가 생성해준다.
> @implSpec 태그를 활성화 하려면 `-tag "implSpec:a:Implementation Requirements"`를 명령줄 매개변수로 지정한다.

#### hook을 잘 선별하여 protected 메서드 형태로 공개하기

```java
/**
 * Removes from this list all of the elements whose index is between
 * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
 * Shifts any succeeding elements to the left (reduces their index).
 * This call shortens the list by {@code (toIndex - fromIndex)} elements.
 * (If {@code toIndex==fromIndex}, this operation has no effect.)
 *
 * <p>This method is called by the {@code clear} operation on this list
 * and its subLists.  Overriding this method to take advantage of
 * the internals of the list implementation can <i>substantially</i>
 * improve the performance of the {@code clear} operation on this list
 * and its subLists.
 *
 * @implSpec
 * This implementation gets a list iterator positioned before
 * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
 * followed by {@code ListIterator.remove} until the entire range has
 * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
 * time, this implementation requires quadratic time.</b>
 *
 * @param fromIndex index of first element to be removed
 * @param toIndex index after last element to be removed
 */
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```
* 해당 메서드는 clear()연산이 이 메서드를 호출함을 보이고 있다.
* clear()를 고성능으로 만들기 용이하도록 이 메서드를 외부로 공개한 것

> **어떤 매서드를 protected로 노출해야할까?**
> 요행은 없지만, 하위 클래스를 만들어 시험해보는 것이 최선이다.
> protected 메서드는 내부 구현에 해당하므로 신중해야한다.
> 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다.
> ex ) 하위 클래스를 만들어보면서 사용 횟수에 따라 protected -> private로 바꾸는 것이 가능하다.


#### 재정의 가능 메서드를 생성자에서 사용하면 안된다.
* `상위 클래스의 생성자`가 `하위 클래스의 생성자`보다 먼저 실행되므로 `하위 클래스의 재정의 메서드`가 `하위 클래스의 생성자`보다 먼저 호출된다.

```java
static class Super {
	public Super() {
		overrideMe();
	}

	public void overrideMe() {
		System.out.println("super method");
	}
}

static class Sub extends Super {
	private final Instant instant;

	Sub() {
		// 부모클래스의 생성자 호출
		instant = Instant.now();
	}

	@Override
	public void overrideMe() {
		System.out.println("sub method");
	}
}

@Test
public void constructorTest() {
	Sub sub = new Sub();
	sub.overrideMe();
}
```

```java
// 출력 결과
null
sub method
```

* 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기 전에 overideMe를 호출한다.
* final 필드의 상태가 두가지다. (원래 final의 상태는 1가지여야함.)
* 위 코드에서 print 는 null도 출력하기 때문에 이상없이 돌아갔지만, 다른 코드에서는 NullPointerException

#### 인터페이스를 사용해서 상속용 설계의 문제점을 해결하기
* 대표적인 예로 Cloneable, Serializable인터페이스가 있다.
* 만약 클래스로 구현한다면 clone(), readObject()는 생성자와 비슷하게 새로운 객체를 만든다.
	* clone : 하위 클래스의 clone 메서드가 복제본의 상태의 수정이 끝나기 전에 재정의 한 메서드를 호출한다. -> 원본 객체까지 피해가 갈 수 있다.
	* readObject : 역직렬화가 끝나기 전에 재정의한 메서드부터 호출한다.
* 따라서, clone과 readObject에서는 재정의 메서드를 호출해서는 안된다.

#### 상속용 클래스의 제약
* 상속용으로 설계하지 않은 클래스는 상속을 금지하기
	* 클래스를 final로 선언하기
	* 모든 생성자를 private || package-private로 선언하고 public 정적 팩터리 만들기
* 재정의 가능 메서드를 사용하는 자기코드를 제거하고 문서로 남기기











