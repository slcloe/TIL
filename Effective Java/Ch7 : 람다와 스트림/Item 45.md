# Item 45 - 스트림은 주의해서 사용하기


#### 스트림의 특징
1. 배열과 같은 시퀀스형 데이터를 처리하는데 특화되어 있다.
2. 스트림 파이프라인은 원소들로 수행하는 연산 단계를 표현하는 개념이다.
	ex. 컬렉션, 배열, 파일, 기본타입 값(int, long, double) 등등 
	**구성** : 소스 스트림 -> 중간 연산 -> 종단 연산 순으로 진행.
3. 스트림의 파이프라인은 지연평가 된다.
4. 평가는 종단 연산이 호출될 때 이뤄진다.
> **스트림에서의 평가는?**
> * 병렬 처리가 가능한 Stream 의 형태에서 Stream 이 아닌 다른 형태의 자바 객체로 바꾸는 행위
> 	* ex . toArray(), collect(), reduce(), forEach()
> * Stream 은 평가 전까지 계속 스트림의 형태를 유지한다.
5. 스트림은 parellel 메서드를 호출하여 병렬로 실행할 수 있지만 효과는 거의 미미하다.


#### 아나그램 예로 살펴보자
**아나그램** 이란?
	* 단어를 구성하는 알파벳은 같고 순서만 다른 단어를 뜻한다.
	* ex. aabc, baac는 아나그램이다.

##### 스트림을 사용하지 않았을 때
```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		File dictionary = new File(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);

		Map<String, Set<String>> groups = new HashMap<>(); 
		try (Scanner s = new Scanner(dictionary)) {
			while (s.hasNext()) { 
				String word = s.next();
				groups.computeIfAbsent(alphabetize(word), 
					(unused) -> new TreeSet<>()).add(word);
			} 
		}
		for (Set<String> group : groups.values()) 
			if (group.size() >= minGroupSize)
				System.out.println(group.size() + ": " + group);
	}
	private static String alphabetize(String s) { 
		char[] a = s.toCharArray(); 
		Arrays.sort(a);
		return new String(a);
	} 
}
``` 
* computeIfAbsent()를 사용하여 맵 안에 키가 있는지 찾은 다음,
	* 있으면, 키에 매핑된 값을 반환
	* 없으면, 함수 객체를 키에 적용하여 값을 계산 후 키-값을 매핑한 다음, 매핑된 값을 반환

##### 스트림을 과하게 사용했을 때
```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);

		try (Stream<String> words = Files.lines(dictionary)) {
			words.collect(
				groupingBy(word -> word.chars().sorted()
							.collect(StringBuilder::new, 
							(sb, c) -> sb.append((char) c), StringBuilder::append).toString()))
			.values().stream()
			.filter(group -> group.size() >= minGroupSize)
			.map(group -> group.size() + ": " + group)
			.forEach(System.out::println); 
		}
	} 
}
``` 
* 스트림을 과용하면 읽기 힘들고, 유지보수도 힘들다.

##### 스트림을 적절히 사용했을 때
```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);
		
		try (Stream<String> words = Files.lines(dictionary)) {
			words.collect(groupingBy(word -> alphabetize(word)))
			.values().stream()
			.filter(group -> group.size() >= minGroupSize) 
			.forEach(g -> System.out.println(g.size() + ": " + g));
		} 
	}
	
	private static String alphabetize(String s) { 
		char[] a = s.toCharArray(); 
		Arrays.sort(a);
		return new String(a);
	}
}
``` 
* 람다에서는 이름을 자주 생략하므로 매개 변수의 이름을 잘 지어야한다.
**유의사항**
* char 값을 처리할 때는 스트림을 삼가는 것이 좋다.
* "Hello world".chars()가 반환하는 스트림의 원소는 int 값이기 때문에 형변환작업을 추가로 요한다.


#### 람다 혹은 스트림보다 코드 블록을 선호하는 경우
* 지역변수를 수정해야할 때
	* 람다에서는 final인 변수만 읽을 수 있고, 지역변수를 수정하는 것은 불가능하다.
* 제어문을 사용할 때
	* return, break, continue처럼 반복문을 제어하는 명령어를 사용하는 것은 코드 블록에서만 가능하다.
* 메서드 선언에 명시된 검사 예외를 던질 때
* 처리 과정 중 이전 단계의 값에 접근해야 하는 경우

#### 람다 혹은 스트림을 사용해야 하는 경우
* 원소들의 시퀀스를 일관되게 변경한다. (map())
* 원소들의 시퀀스를 필터링한다. (filter())
* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다. (collect)
* 원소들의 시퀀스를 컬렉션에 모은다. (collect)
* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

#### 메르센 소수 예제로 살펴보자
**메르센 소수** 란?
	* 2^p-1를 메르센 수라고 하고 메르센 수 중 소수인 수를 메르센 소수라고 한다.

```java
static Stream<BigInteger> primes() {
	return Stream.iterate(TWO, BigInteger::nextProbablePrime); 
}

public static void main(String[] args) {
	primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
			.filter(mersenne -> mersenne.isProbablePrime(50))
			.limit(20)
			.forEach(System.out::println);
}
```
* 10개의 메르센 소수를 구하는 코드
* 만약 p를 알고 싶다면?
	* 스트림의 초기 값은 종단 연산에서는 접근할 수 없다.
	* 하지만 역으로 계산하여 p를 구할 수 있다.
	`.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));`

#### 데카르트 예제로 살펴보자
* 반복문 구현
```java
private static List<Card> newDeck() { 
	List<Card> result = new ArrayList<>(); 
	for (Suit suit : Suit.values())
		for (Rank rank : Rank.values()) 
			result.add(new Card(suit, rank));
	return result; 
}
``` 
* 스트림 구현
```java
private static List<Card> newDeck() { 
	return Stream.of(Suit.values())
		.flatMap(suit -> Stream.of(Rank.values())
		.map(rank -> new Card(suit, rank))) .collect(toList());
}
``` 
* 스트림과 반복 중 어느 족이 나은지 선택하는 규칙은 없지만, 더 이해하고, 유지보수하기 편한 쪽으로 선택해라.

<!-- 
```java

``` 
-->