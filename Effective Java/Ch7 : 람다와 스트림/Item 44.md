# Item 44 - 표준 함수형 인터페이스를 사용하기

```java
// LinkedHashMap의 내부 메서드

//1. 재정의
protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
  return size() > 100;
}

// 함수인터페이스 사용
@FunctionalInterface
interface EldestEntryRemovalFunction<K, V> {
  boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}

// 표준 함수형 인터페이스 사용
BiPredicate<Map<K, V>, Map.Entry<K, V>>
``` 
* 필요한 용도에 맞는게 있다면, 직접 구현 보다는 표준 함수형 인터페이스를 활용하자

#### 주요 6개의 함수형 인터페이스 기본형
* UnaryOperator<T>, T apply(T t), String::toLowerCase
	* 인자가 1개인 메서드
	* 반환값과 인수의 타입이 같다.
* BinaryOperator<T>, T apply(T t1, T t2), BigInteger::add
	* 인자가 2개인 메서드
	* 반환값과 인수의 타입이 같다.
* Predicate<T>, boolean test(T t1), Collection::isEmpty
	* 인자가 1개인 메서드로 
	* boolean을 반환한다.
* Function<T, R>, R apply(T t), Arrays::asList
	* 인자가 1개인 메서드로 
	* 반환값과 인수의 타입이 다르다.
* Supplier<T>, T get(), Instant::now
	* 인자가 없는 메서드
* Consumer<T>, void accept(T t), System.out::println
	* 인자가 1개인 메서드
	* 반환값은 없다.

#### 기본형을 반환하는 함수 인터페이스의 변형
* SrcToResult
	* long을 받아 int를 반환 -> LongToIntFunction
* ToResult
	* int[] 인수를 받아 long 을 반환 -> ToLongFunction<int[]>

**유의**
* 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지 말기
	* 계산량이 많을 때 성능이 느려질 수 있다.

#### @FunctionalInterface의 사용
* 직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 를 사용해라.
	1. 해당 클래스의 코드, 문서에 람다용으로 설계됨 인것을 알려줘야한다.
	2. 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일 되게 해준다.
	3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

### 함수형 인터페이스 사용시 주의점
* 서로 다른 험수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 오버로드 하지말자.
	* 올바른 메서드를 알려주기 위한 형변환이 필요할 수 있다.


```java

``` 


```java

``` 
<!-- 
```java

``` 
-->