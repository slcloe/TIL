# Item 37 - ordinal 인덱싱 대신 EnumMap을 사용해라

#### ordinal 메서드를 사용한 예제
```java
class Plant {
	enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

	final String name;
	final LifeCycle lifeCycle;

	Plant(String name, LifeCycle lifeCycle) { 
		this.name = name;
		this.lifeCycle = lifeCycle; 
	}

	@Override public String toString() {
		return name; 
	}
}
``` 

```java
Set<Plant>[] plantsByLifeCycle =
	(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++) 
	plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
	plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
	System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
``` 
**문제점**
1. 배열은 제네릭과 호환이 되지 않기 때문에 비검사 형변환을 수행해야함.
2. 정확한 정수값을 사용하는지 스스로 보증해야한다.
	만약 정수값이 잘못된다면? -> throw ArrayIndexOutOfBoundsException

#### EnumMap을 사용한 예제

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = 
	new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) 
	plantsByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden) 
	plantsByLifeCycle.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
``` 
1. 코드가 더 간결하고 성능도 비등하다.
	EnumMap의 내부 구현 방식에서 배열을 사용한다.
2. 제네릭 타입을 사용하기 때문에 컴파일 경고 없이 타입 안정성을 보장한다.


#### 스트림 사용 예제
```java
System.out.println(Arrays.stream(garden)
	.collect(groupingBy(p -> p.lifeCycle)));

System.out.println(Arrays.stream(garden) 
	.collect(groupingBy(p -> p.lifeCycle,
		() -> new EnumMap<>(LifeCycle.class), toSet())));
``` 
* EnumMap버전에서는 맵을 무조건 N개(종류별) 만들었지만, 스트림 버전에서는 필요한 그룹만 만든다.


#### ordinal()로 2차원 배열 인덱스 구현하기
```java
enum Phase {
    SOLID, LIQUID, GAS;

    enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL}, 
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
``` 
* 컴파일러는 orinal과 배열 인덱스의 관계를 모른다.
	* 열거 타입이 수정될 때 TRANSACTIONS를 제대로 수정해주지 않으면 런타임에러를 야기한다.
		( 컴파일 상으론 문제가 없기때문에 컴파일 에러는 일어나지 않는다.)
	* NullPointerException, IndexOutOfBoundsException을 야기할 수도 있다.
* 테이블이 커질수록 null의 공간이 많아져 공간 낭비를 초래한다.


#### EnumMap으로 2차원 배열 인덱스 구현하기
```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private final static Map<Phase, Map<Phase, Transition>> map =
                Stream.of(values()).collect(groupingBy(
                        t -> t.from,
                        () -> new EnumMap<>(Phase.class),
                        toMap(t -> t.to, t -> t,
                                (x, y) -> y,
                                () -> new EnumMap<>(Phase.class)
                        )
                ));

        public static Transition from(Phase from, Phase to) {
            return map.get(from).get(to);
        }
    }
}
``` 
* IONIZE, DEIONIZE를 새로 추가해도 Phase에 PLASMA를 추가하고 Transaction에 전이관계를 추가하면 된다.
	* ordinal로 구현하면 이상의 공간비용을 요구하고, 잘못된 순서로 나열할 경우 런타임 문제를 일으킨다.
* 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표기하자.

<!-- 
```java

``` 
-->