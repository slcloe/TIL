 # Item 15 - 클래스와 멤버의 접근 권한을 최소화하자

 잘 설계된 컴포넌트란?
 구현(private)과 API(public)가 명확히 분리되어 있다!

 #### 정보 은닉의 장점
 1. 개발 속도 향상
	여러 컴포넌트를 병렬로 개발할 수 있다.
 2. 관리 비용 감소
	컴포넌트를 빨리 파악해서 디버깅 할 수 있다.
 3. 성능 최적화 (성능 자체는 X)
	다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트의 최적화에 집중 가능
 4. SW 재사용성 높임
	독자적으로 동작할 수 있는 컴포넌트 -> 낯선 환경에서도 사용가능
 5. 개발 난이도 낮춤
	개별 컴포넌트의 동작을 검증할 수 있음

#### SW가 올바르게 동작하는 한 항상 가장 낮은 접근 수준을 부여해야한다.

1. package-private 로 선언하기
	톱레벨 클래스/인터페이스에 부여할 수 있는 접근 수준 : package-private & public
	* public : 공개 API일 때 -> 하위 호환을 위해 영원히 관리해야함
	* protected : 공개 API
	* package-private : 내부 구현일 때 -> 언제든 수정 가능
	* private : 내부 구현

2. 클래스 안에 private static 으로 중첩시키기
3. 테스트만을 위한 클래스, 인터페이스, 멤버를 공개API로 만들면 안된다.
	* 테스트 코드와 테스트 대상을 같은 패키지에 두면 package-private 요소에 접근 가능
4. public 클래스의 인스턴스 필드는 public을 피해라
	* public 가변필드는 `불변함을 보장할 수 없다` ,  `thread-safe 하지 않다.`
	* public 불변필드는 `코드 수정 시 public 필드를 없애는 방식으로는 리팩토링 불가능`

5. 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공하지 말기
	* 클라이언트에서 그 배열의 내용을 수정할 수 있기 때문!

	```java
	// 문제 코드
		public static final Thing[] VALUES = { ... };
	```


	```java
	// 해결책1 - public 불변 리스트 추가
		private static final Thing[] PRIVATE_VALUES = { ... }; // public -> private으로 변경
		public static final List<Thing> VALUES = 
			Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
	```


	```java
	// 해결책 2 - public 복사 메서드 추가
		private static final Thing[] PRIVATE_VALUES = { ... }; // public -> private으로 변경
		public static final Thing[] values() {
			return PRIVATE_VALUES.clone();
		}

	```
