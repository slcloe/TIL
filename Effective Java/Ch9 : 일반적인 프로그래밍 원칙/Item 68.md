# Item 68 - 일반적으로 통용되는 명명 규칙 따르기
## 철자 
#### 패키지
* 각 요소를 .으로 구분하여 계층적으로 이름을 짓는다.
* 인터넷 도메인 이름을 역순으로 사용한다.
* 예외. 표준 라이브러리와 선택적 패키지들은 각각 java, javax 로 시작한다.
* 각 요소는 8자 이하의 짧은 단어로 한다.
	* 때에 따라 약어를 사용하기도 한다.

#### 클래스와 인터페이스
* 하나 이상의 단어로 이루어지고, 각 단어는 대문자로 시작한다.
* 통용되는 줄임말 외에는 단어를 줄여쓰지 않는다. (min, max)
* 약자에 경우는 첫글자만 대문자로 하는 쪽이 더 많다.	
	* HttpUrl vs HTTPURL

#### 메서드와 필드
* 첫 글자를 소문자로 쓴다.
* 이외는 클래스의 명명 규칙과 같다.

#### 상수 필드
* 모두 대문자로 쓰며, 단어 사이는 모두 밑줄로 구분한다.
* static final, 불변 참조 타입도 이에 해당된다.

#### 지역 변수
* 약어를 써도 좋다.
* 입력 매개변수도 지역변수로 본다.

#### 타입 매개변수
* 한 문자로 표현한다.
* T: 임의의 타입
* E: 컬렉션 원소의 타입
* K, V: 맵의 키와 값
* X: 예외
* R: 메서드의 반환 타입
* T, U, V, T1, T2, T3: 임의의 타입 시퀀스

## 문법
#### 클래스와 인터페이스
* 객체를 생성할 수 있는 클래스라면 단수 명사나 명사구를 사용한다.
	* ex. Thread, PriorityQueue, ChessPiece
* 객체를 생성할 수 없는 클래스라면 복수형 명사를 사용한다.
	* ex. Collectors, Collections
* 인터페이스 이름은 able, ible로 끝나는 형용사를 쓰거나 클래스와 똑같이 짓는다
	* ex. Runnable, Iterable, Accessible, Collection, Comparator

#### 에너테이션
* 다른 규칙이 없다.
	* ex. BindingAnnotation, Inject, ImplementedBy, Singleton

#### 메서드
* 동작을 수행하는 메서드는 동사, 동사구로 짓는다.
	* ex. append, drawImage
* boolean 값을 반환하는 메서드는 is, has로 시작한다. 끝은 아무 단어나 구로 끝나도록 짓는다.
	* ex. isDigit, isProbablePrime, isEmpty, isEnabled, hasSiblings
* boolean 값을 반환하지 않거나 속성을 반환하는 메서드는, 명사, 명사구, get으로 시작하는 동사구로 짓는다.
	* ex. size, hashCode, getTime
* 객체 타입을 바꿔서 다른 객체를 반환하는 메서드는 toType형태로 짓는다.
	* ex. toString, toArray
* 객체의 내용ㅇ르 다른 뷰로 보여주는 메서드는 asType형태로 짓는다.
	* ex. asList
* 객체의 값을 기본 타입 값으로 반환하는 메서드는 typeValue 형태로 짓는다.
	* ex. intValue
* 정적 팩터리는 from, of, valueOf, instance, getInstance, newInstance, getType, newType을 사용한다.

#### 필드 
* 클래스, 인터페이스, 메서드 이름에 비해 덜 명확하고, 덜 중요하다.
* API 설계를 잘 했다면 필드명이 직접 노출될 일이 거의 없다.

<!--
```java

```
 -->