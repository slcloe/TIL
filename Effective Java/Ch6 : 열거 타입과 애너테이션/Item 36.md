# Item 36 - 비트 필드 대신 EnumSet을 사용하기

#### 비트 필드 열거 상수

```java
public class Text {
	public static final int STYLE_BOLD = 1 << 0; // 1
	public static final int STYLE_ITALIC = 1 << 1; // 2
	public static final int STYLE_UNDERLINE = 1 << 2; // 4
	public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

	public void applyStyles(int styles) { ... }
}
```

```java
// 사용 예시
	text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

**장점**
합집합이나 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

**단점**
1. 정수 열거 상수의 단점을 똑같이 가지고 있다.
2. 모든 원소를 순회하기 힘들다.
3. 최대 몇 비트가 필요한지 API 작성 시 미리 예측해야한다.
	API를 수정하지 않고서는 비트 수를 더 늘릴 수 없기 때문.

#### EnumSet 사용하기
```java
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
	// 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
	public void applyStyles(Set<Style> styles) { ... } 
}
```
* EnumSet의 내부는 비트 벡터로 구현되었다.
* 원소가 총 64개 이하라면 RegularEnumSet을 사용하여 비트필드에 비견되는 성능을 보여준다.
* 그 이상은 JumboEnumSet을 사용한다. : 크기에도 유연하다.

```java
// 사용 예시
	text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

* 불변 EnumSet이 필요하다면 Collections.unmodifiableSet을 활용하자.


<!--
```java

```
-->