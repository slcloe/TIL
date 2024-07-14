# Item 46 - 스트림에서는 부작용 없는 함수를 사용하기

### 스트림을 잘 활용하기 위한 방법

##### 이해하지 못한 채 스트림을 썼을 때
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
words.forEach(word -> {
freq.merge(word.toLowerCase(), 1L, Long::sum); });
}
``` 
* for-Each 는 스트림이 수행한 연산 결과를 보여주는 일만 수행해야하지만
* 위 코드에서는 람다에서 상태를 수정한다. -> 나쁜 코드
###### Stream 내부에서 사이드 이팩트를 사용하면 발생하는 문제
	* 사이드 이펙트 : 의도하지 않은 결과 혹은 결과 값에 미치지 않는 모든 작업을 의미 
	1. 가독성 : Stream을 사용할 때 변환, 평가가 이루어질 것 같지만 그런 코드가 아니다.
	2. 재사용성 : 외부 상태에 의존하기 때문에 재사용할 수 없다.
	3. 테스트 가능성 : Stream은 병렬로 실행될 수 있기 때문에 테스트 하기 힘들어진다.
	4. 동시성 : 공유 가변 상태와 같이 공유 자원에 관련된 문제가 일어날 수 있다.

##### 스트림을 활용해 빈도표를 초기화
```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
freq = words
.collect(groupingBy(String::toLowerCase, counting()));
}
``` 
* for-each 연산은 스트림의 계산 결과를 보고할 때만 사용해야한다. ( 종단 연산에만 사용하자 )
* 하지만 위 코드에서는 계산하는데 쓰고 있기 때문에 적절한 코드가 아니다.

##### 적절한 스트림 활용법
```java
List<String> topTen = freq.keySet().stream().sorted(comparing(freq::get).reversed()) .limit(10)
.collect(toList());
``` 
* `comparing(freq::get).reversed()`
	* comparing : 키 추출 함수를 받는 비교자 생성 메서드
	* freq::get : 입력받은 단어(키)를 빈도표에서 찾아(추출) 그 빈도를 반환함
	* reserved() : 가장 빈도가 많은 단어가 첫번째로 오도록 정렬한다.

#### Collector API

#### toMap (인수 2개)
```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(
toMap(Object::toString, e -> e));
``` 
* 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.
* 스트림 원소 키가 충돌된다면 IllegalStateException을 던진다.


#### toMap (인수 3개)
```java
Map<Artist, Album> topHits = albums.collect(
toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
``` 
* 첫번째 인자 - Key, 두번째 인자 - value,
* 세번째 인자 - 선택할 값의 로직
* 위 예시에서는 세번째 인자에 비교자가 들어갔기 때문에 Album의 sales 가 가장 많은 값이 선택될 것이다.
```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
``` 
* 이 예시에서는 가장 마지막에 입력된 값이 선택될 것이다.

#### toMap (인수 4개)
* 네번째 인수로 맵 팩터리를 받는다.
* EnumMap, TreeMap 처럼 원하는 특정 맵의 구현체를 직접 지정할 수 있다.
</br>

* toConcurrentMap : 병렬 실행된 후 결과로 ConcurrentHashmap 인스턴스를 생성한다.

#### groupingBy()
```java
words.collect(groupingBy(word -> alphabetize(word)))
``` 
* 아나그램 프로그램에서 key값을 기준으로 grouping 하여 보여준다.
* groupingByConcurrent : 병렬 실행 버전으로 ConcurrentHashMap 인스턴스를 생성한다.
* 비슷하게 partitioningBy() 도 있다.
	boolean 인 맵을 반환한다.
```java
Map<Boolean, List<String>> partitioningByMap = 
	words.stream() 
	.collect(partitioningBy((word) -> word.equals("stop")));
System.out.println("partitioningByMap = " + partitioningByMap);
``` 
* stop 은 true 로 나머지는 false로 결과값이 도출된다.

#### 숫자 집계 메서드들 
	int sumOfScore = students.stream()
    					.collect(summingInt(Student::score));
    double averageOfScore = students.stream()
                                    .collect(averagingInt(Student::score));
    IntSummaryStatistics intSummaryOfScore = students.stream()
                                                     .collect(summarizingInt((s) -> s.score));

#### Joining
* joining(delimeter)
```java
List<String> food = new ArrayList<>();
food.add("피자");
food.add("햄버거");
food.add("치킨");

String collect = food.stream().collect(joining(" 그리고 "));
System.out.println("collect = " + collect);

collect = 피자 그리고 햄버거 그리고 치킨
``` 

* joining(delimeter, prefix, suffix)
```java
List<String> food = new ArrayList<>();
    food.add("피자");
    food.add("햄버거");
    food.add("치킨");

String collect = food.stream().collect(
		joining(", ", "맛있는 ", "이 좋아"));

System.out.println("collect = " + collect);
collect = 맛있는 피자, 햄버거, 치킨이 좋아
``` 

<!-- 
```java

``` 
-->