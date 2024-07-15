# Item 43 - 람다보다는 메서드를 참조를 사용하기

#### 메서드 참조를 사용하기
* 매개 변수가 늘어날 수 록 메서드 참조로 제거할 수 있는 코드 양도 늘어난다.
* 하지만, 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다.
	* 예외) 함수 타입이 제네릭일 때는 예외이다.
* 이름을 지어서 적절한 설명을 남길 수 있다. (람다는 불가능함)


###### 메서드 참조 사용 전
```java
map.merge(key, 1, (count, incr) -> count + incr);
``` 

###### 메서드 참조 사용 후
```java
map.merge(key, 1, Integer::sum);
```

* 보통은 간결하지만 예외인 예시가 있다.

* 메서드와 람다가 같은 클래스에 있을 때,
```java
service.execute(GoshThisClassNameIsHumongous::action);
``` 
```java
service.execute(() -> action())
``` 

#### 메서드 참조의 유형
정적: `Integer::parseInt` == `str -> Integer.parseInt(str)`  
한정적 (인스턴스): `Instant.now()::isAfter` == `Instant then = Instant.now(); t -> then.isAfter(t)`  
비한정적 (인스턴스): `String::toLowerCase` == `str.toLowerCase()`  
클래스 생성자: `TreeMap<K, V>::new `== `() -> new TreeMap<K, V>()`  
배열 생성자: `int[]::new` == `len -> new int[len]`  


-> 메서드 참조와 람다 중 훨씬 코드를 작성하는 편이 짧고 명확한 쪽으로 사용하자.!

<!-- 
```java

``` 
-->
