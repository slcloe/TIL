# Item 32 - 제네릭과 가변인수를 함께 쓸 때는 신중하기

#### 가변인수 메서드의 위험성
* 가변인수 메서드를 호출하면 가변 인수를 담기 위한 배열이 자동으로 하나 만들어진다.
* 이 배열을 클라이언트에게 공개가 되면 문제가 발생한다.
-> 메서드를 선언할 때 실체화불가타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다.

```java
public static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // 힙 오염 발생
    String s = stringLists[0].get(0); // throw ClassCastException
}
```
* 마지막 줄에서 형변환이 일어나는데, 이때 타입 안정성이 깨지게 되고, 제니릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

```java
public static <T> T[] toArray(T... args) {
    return args;
}
``` 
* 메서드가 반환하는 배열의 타입은 컴파일타임에 결정되는데, 하지만 그 시점에서는 컴파일러는 T가 어떤 자료형인지 알 수 없기 때문에 타입을 잘못 판단할 수 있다.
* 이 상태에서 가변 매개변수 배열을 그대로 반환하면 힙 오염이 전이될 수 있다.

```java
public static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }

    throw new AssertionError(); // 도달할 수 없다.
}
``` 
* 가변 제네릭 인자를 변환한 toArray()의 결과를 이용하는 점이 위험하다.
* toArray의 반환형은 Object[]이며 T == Object 가 아니라면 형변환은 실패한다.
###### 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다.


#### 제네릭 가변인수 사용
1. @safeVarargs의 선언된 메서드에 넘기는 것은 안전하다.
2. varargs를 받지 않는 일반 메서드에 넘기는 것은 안전하다.
3. 제네릭 가변인수를 외부로 노출시키지 않았다.
4. 제네릭 가변인수에 어떤것도 저장하지 않는다.
```java
@SafeVarargs
static <T> List<T> flatten(List <? extends T>... lists) {
    List<T> result = new ArrayList<>();

    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
``` 
> `@SafeVarargs`
> 재정의 할 수 없는 메서드에만 달아야한다.
> 이후 재정의한 메서드가 안전한지는 보장할 수 없기 때문이다.!


```java
static <T> List<T> flatten(List<List <? extends T>> lists) {
    List<T> result = new ArrayList<>();

    for(List<? extends T> list : lists) {
        result.addAll(list);
    }

    return result;
}
``` 
* varargs 매개변수를 List로 대체할 수 있다.

<!-- 
```java

``` 
-->