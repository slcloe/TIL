# Item 26 - raw type은 사용하지 말기

* 클래스와 인터페이스 선언에 타입 매개변수가 쓰이면 이를 제네릭 클래스 혹은 제네릭 인터페이스 라고 한다.
	* 통틀어 제네릭 타입이라고도 함.
* 각각의 제네릭 타입은 일련의 매개변수화 타입을 정의한다.
* 제네릭 타입을 정의하면 그에 딸린 raw type도 함께 정의된다.
	* raw type : List, Set
	* raw type은 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.
	* 제네릭을 지원하기 전에 사용했던 것이라 문제가 많다.

#### raw type의 문제점
```java
private final Collection stamps = ...;

stamps.add(new Coin(...));

for (Iterator it = stamps.iterator(); i.hasNext();) {
  Stamp stamp = (Stamp) i.next(); // throw ClassCastException
  stamp.cancel();
}
```
* 실수로 stamp 가 아닌 coin 을 넣어도 아무 오류 없이 컴파일 된다.
* 런타임 시에는 throw ClassCastException을 던지게 된다.
	* 문제를 겪는 코드와 원인 코드가 물리적으로 떨어져 있어 해결하기 힘들다.

```java
private final Collection<Stamp> stamps = ...;
```
 
* 위와 같이 작성 시 stamp 가 아닌 다른 인스턴스를 삽입해도 컴파일 오류가 발생한다.
* 컴파일러는 보이지 않는 형변환을 추가하여 절대 실패하지 않음을 보장한다.
* 그럼에도 불구하고 raw-type이 존재하는 이유는 하위버전의 호환성 때문이다.


#### `List<Object>`와 List의 차이
* `List<Object>` 는 괜찮지만, List는 사용하면 안된다.
* `List<Object>` 는 제네릭에 명확히 Object의 하위 타입을 받겠다고 선언했지만,
* `List` 는 제네릭을 포기한것이다.

##### List raw-type인 경우
```java
public static void main(String[] args){
	List<String> strings = new ArrayList<>();
	unsafeAdd(strings, Integer.valueOf(42));
	String s = strings.get(0);
}

private void unsafeAdd(List list, Object o) {
	list.add(o);
}
```
* 컴파일은 가능하다.
* 하지만, 런타임시 에러가 발견된다.
> Test.java:10: warning: [unchecked] unchecked call to add(E) as a member of the raw type List
> list.add(o);

##### `List<Object>` 제네릭인 경우

```java
public static void main(String[] args){
	List<String> strings = new ArrayList<>();
	unsafeAdd(strings, Integer.valueOf(42));
	String s = strings.get(0);
}

private void unsafeAdd(List<Object> list, Object o) {
	list.add(o);
}
```
* 컴파일 에러가 발생한다.
> Test.java:5: error: incompatible types: `List<String>` cannot be converted to `List<Object>`
> unsafeAdd(strings, Integer.valueOf(42));

#### 모든 타입을 받을 수 있는 비한정 와일드 카드 타입
* `List<Object`대신 모든 타입을 받을 수 있는 비한정 와일드 카드 타입을 사용하기
```java
static int num(Set<?> s1, Set<?> s2)
```
* null 이외에 어떤 원소도 넣을 수 없다.
* 이 제약에서 벗어나려면 `? extends 클래스`와 같이 `한정적 와일드 카드 타입`을 사용해라
* 아무 원소나 넣을 수 없으니 타입 불변식을 훼손하기 어렵다.


#### raw-type을 써야할 때

##### class 리터럴을 사용할 때
* 자바 명세에  class 리터럴에 매개변수화 타입을 사용하지 못하게 했다. (배열, 기본타입은 허용한다.)
ex ) List.class, String[].class, int.class는 허용하지만, `List<String>.class` 는 허용하지 않는다.

##### instanceof 연산자를 사용할 때
* 런타임에는 제네릭 타입 정보가 지워지기 때문에 instanceof 연산자는 비한정적 와일드카드 타입`List<E>` 이외의 매개변수화 타입`List<String>`에는 적용할 수 없다.

```java
if (o instanceof Set) {  // 로 타입
  Set<?> s = (Set<?>) o; // 와일드카드 타입
  ...
}
```