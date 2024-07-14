# Item 48 - 스트림 병렬화는 주의해서 사용하기


#### 자바 언어와 동시성
* 자바는 첫 릴리즈부터 스레드, 동기화, wait/notify를 지원했다.
* java.util.concurrent 라이브러리와 실행자 프레임 워크 지원
* 고성능 병렬 분해 프레임 워크인 fork-join 패키지를 추가
* parellel 메서드로 병렬 실행 할 수 있는 스트림을 지원

#### parallel()의 문제점 
```java
public static void main(String[] args) {
primes()
.parallel()
.map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
.filter(mersenne -> mersenne.isProbablePrime(50)) .limit(20)
.forEach(System.out::println);
}
static Stream<BigInteger> primes() {
return Stream.iterate(TWO, BigInteger::nextProbablePrime); }
``` 
* parallel을 호출했지만, 연산이 끝나지 않고 계속된다.
* 스트림라이브러리가 파이프라인을 병렬화 하는 법을 찾아내지 못했기 때문!
=> 데이터 소스가 stream.iterate거나 종간 연산으로 limit를 쓰면 파이프라인 병렬화를 통한 성능개선은 불가능하다.
	* limit()가 있을 때, CPU코어가 남는다면 원소 몇개를 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다.
	* 계속 버려지기 때문에 계속 이전까지의 값을 다시 구해야한다. 

#### parallel() 사용법
* 스트림의 소스가 `ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap`의 인스턴스거나, `배열`, `int 범위`, `long 범위` 등 쪼개기 쉬울 때 병렬화의 효과가 가장 좋다.
	* 위 자료구조의 공통점
		1. 데이터를 원하는 크기로 정확하게 손쉽게 나눌 수 있어 쓰레드에 분배하기 졶다.
			* 나누는 작업은 Spliterator가 담당한다.
			* Spliterator의 객체는 Stream, Iterable의 spliterator메서드로 얻을 ㅜㅅ 있다.
		2. 순차적으로 실행할 때 참조 지역성이 뛰어나다.
			* 하지만, 실제 객체가 메머리에서 떨어져 있다면 참조 지역성이 나쁘다.
			* 참조 지역성이 낮으면 쓰레드는 데이터가 주 메모리->캐시메모리 로 로딩될 때까지 대기해야한다.
			* 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화 할 때 중요한 요소다.
			* 참고로, 참조 지역성이 가장 뛰어난 자료구조는 `기본타입 배열`이다.

* 종단 연산
	* 파이프 라인의 종단 방식은 병렬 수행 효율에 영향을 준다.
	* 축소(reduction)
		(파이프 라인에서 만들어진 모든 원소를 하나로 합친다.)
	* 완성된 형태로 제공하는 메서드 
		ex. max, min, count, sum...
	* 조건에 맞으면 바로 반환되는 메서드
		ex. anyMatch, allMatch, noneMatch
	* 하지만, 가변 축소를 수행하는 Stream 의 collect는 병렬화에 적합하지 않다.

##### 스트림을 잘못 병렬화 하면?
* 성능 저하 / 결과 오류 등 예상치 못한 동작이 발생할 수 있다.
	* **안전실패** : 결과가 잘못되거나 오동작하는 것.
	* 연산의 요구사항을 지키지 못해도 파이프라인을 순차적으로 수행한다면 바른 결과르 얻을 수 있다.
	* 하지만, 이 상태로 병렬로 수행하기 된다면 실퍠로 이어질 수 있다.


#### 스트림 병렬화 적용

* 스트림 병렬화는 오직 성능 최적화 수단이다.
* 변경 전후로 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지 확인해야 한다.

```java
static long pi(long n) {
return LongStream.rangeClosed(2, n)
.parallel() .mapToObj(BigInteger::valueOf) .filter(i -> i.isProbablePrime(50)) .count();
}
``` 
* 이 작업은 쪼개기 쉽기 떄문에, parellel() 메서드로 약 5배 성능 향상을 이룰 수 있다.

* 만약 무작위 수로 이뤄진 스트림을 병렬화 하고 싶다면?
	* ThreadLocalRandom : 단일 쓰레드에서 쓰기 위해 만들어짐
	* SplittableRandom : 무작위 수들로 이뤄진 스트림을 병렬화하기 위해 만들어짐 (성능이 선형으로 증가.)
	* Random : 모든 연산을 동기화하기 때문에 병렬 처리하면 최악의 성능.

<!-- 
```java

``` 
-->