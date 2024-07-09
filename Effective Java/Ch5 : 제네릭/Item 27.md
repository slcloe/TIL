# Item 27 - 비검사 경고를 제거하기

#### 비검사 경고가 나타날 때

```java
// 비검사 경고가 발생하는 코드의 예시
Set<Lark> exaltation = new HashSet();
```

```java
// 컴파일러 경고
Venery.java:4: warning: [unchecked] unchecked conversion
        Set<Lark> exaltation = new HashSet();

required: Set<Lark>
found: HashSet
```
* 자바 7부터는 `<>` 다이아몬드 연산자를 사용하면 자동으로 타입을 추론 해준다.
* 최대한 모든 비검사 경고를 제거해야 한다.
	1. 타입 안정성 보장
	2. ClassCastException 을 피할 수 있다.

#### 비검사 경고를 제거할 수 없을 떼

* @SuppressWarnings 를 사용하여 경고를 숨기기
	* 하지만, 항상 가능한 한 좁은 범위에 적용하자
	ex ) 변수 선언, 짧은 메서드, 생성자 ( 클래스 전체에는 사용하지말것)

```java
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
    return (T[]) Arrays.copyOf(elements, size, a.getClass());
  }
  System.arraycopy(elements, 0, a, 0, size);

  if(a.length > size) {
    a[size] = null;
  }

  return a;
}
```
* 위 코드는 return 시에 배열 형변환에서 경고를 던진다.
```java
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
    // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
    @SuppressWarnings("unchecked")
    T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }

  System.arraycopy(elements, 0, a, 0, size);

  if(a.length > size) {
    a[size] = null;
  }

  return a;
}
```
* return문에는 @SupressWarnings를 달 수 없다.
* 메서드에 달기엔 범위가 너무 넓다.
* 변수를 따로 추가하여 @SupressWarnings를 달았다.
* @SuppressWarnings("unchecked")를 사용할 때는 그 경고를 무시해야하는 안전한 이유를 항상 주석으로 남겨야한다.
<!-- 
```java

``` -->
