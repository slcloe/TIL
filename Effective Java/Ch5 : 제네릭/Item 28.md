# Item 28 - 배열보다는 리스트를 사용하기



#### 배열과 리스트의 차이점
1. 공변과 불공변
* 배열 : 공변 ( 같이 변한다. )
* 리스트(제네릭) : 불공변 ( 같이 변하지 않는다. )

```java
Object[] objects = new Long[1];
objects[0] = "String"; // throw ArrayStoreException
```
* 배열은 공변이라 컴파일은 되지만, 런타임시 실패한다.

```java
List<Object> ol = new ArrayList<Long>(); // 불공변이라 타입 자체가 호환이 안된다. 이미 에러가 뜨고 있다.
    ol.add("힝냐");
```
* 리스트는 불공변이라 컴파일도 되지 않는다.

##### 에러는 컴파일 시에 잡는 것이 가장 이상적이다.


2. 실체화의 유무
* 배열 : 런타임 시에도 자신기 담기로 한 원소의 타입을 인지하고 확인한다.
	* 실체화 가능
* 리스트(제네릭) : 타입 정보가 컴파일에서 확인 후, 런타임 시에는 소거된다.
	-> new List<E>[], new List<String>[], new E[]는 컴파일 시 제네릭 배열 생성 오류를 일으킨다.
	* 실체화 불가능

##### 배열
```java
1. List<String>[] stringLists = new List<String>[1]; // 컴파일 에러가 나야할 곳
2. List<Integer> intList = List.of(42);
3. Object[] objects = stringLists;
4. objects[0] = intList;
5. String s = stringLists[0].get(0); // throw ClassCastException
``` 
* 1번에서 컴파일 에러가 나지 않으면, 5번에서 런타임 시 throw ClassCastExpcetion이 일어나게 된다.
* 오류발생지와 원인문장이 너무 떨어져 있어 오류를 해결하기 힘들다.



##### 제네릭
* E, `List<E>`, `List<String>`같은 타입을 실체화 불가 타입이라고 한다.
* 컴파일시점보다 런타임 시에 타입 정보를 적게 가진다.
* 소거 매커니즘 때문에, 매개변수화 타입 가운데 실체화될 수 있는 타입은 `List<?>`와 `Map<?, ?>` 같은 비한정적 와일드카드 타입 뿐이다.
<!-- 소거 메커니즘이란? -->

> 배열로 형변환 할 때,
> 제네릭 배열 생성 오류 || 비검사 형변환 경고가 뜨는 경우가 있다.
> 대부분은 E[] 대신 `List<E>`를 쓰면 해결된다.
> 코드의 복잡성이 올라가고 성능저하가 있지만, 타입 안전성과 상호운용성은 좋아진다.

#### 제네릭으로 코드 개선하기

```java
// 제네릭을 사용하지 않고 있다.
public class Chooser {
  private final Object[] choiceArray;

  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
``` 
* choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다.
* 만약, 다른 원소가 들어 있었다면 런타임시에 ClassCastException이 날 것이다.


```java
// 제네릭 적용
public class Chooser<T> {
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    choiceArray = choices.toArray();
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
``` 
* Collection.toArray()의 반환형은 Object[] 이기 때문에 항상 (T[])로 캐스팅을 해줘야한다.
* 이후 고친다 하더라도 unchecked cast 경고가 뜬다. 
	-> 이유는 런타임에는 제네릭 타입 정보가 소거 되었기 때문에 컴파일러가 안전성을 보장하지 못할 뿐 잘 돌아간다.
	-> 주석과 @를 달 것!


```java
// 리스트 적용
public class Chooser<T> {
  private final List<T> choiceList;

  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<T>(choices);
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceArray.length));
  }
}
``` 
* 제네릭을 통해 타입 안정성을 확보했다. 따라서, 오류나 경고 없이 컴파일된다.
* 공변이 더 위험할 수도 있다.

<!-- 
```java


``` 
-->