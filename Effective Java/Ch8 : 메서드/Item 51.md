# Item 51 - 메서드 시그니처를 신중히 설계하기

### 메서드 이름을 신중히 짓자.
* 같은 패키지에 속한 이름들과 일관되게 짓자.
* 긴 이름은 피하자.
* 개발자 커뮤니티를 참고하자.

### 편의 메서드를 너무 많이 만들지 말자
* 메서드가 너무 많으면 다른 개발자가 사용하기도 힘들고, 문서화, 유지보두고 힘들다.
* 클래스나 인터페이스는 각 기능을 완벽히 수행하는 메서드로 제공해야 한다.

### 매개변수 목록을 짧게 유지하자.
* 일반적으로 4개 이하가 좋다.
* 같은 타입의 매개변수가 여러개 연달아 나오는 경우는 헷갈리기 때문에 좋지 않다.
	* 실수로 순서를 바궈 입력해도 컴파일되어 실행되기 때문이다.


### 긴 매개변수 목록을 짧게 줄이는 방법
1. 여러 메서드로 쪼개기
* 직교성을 높여 메서드 수를 줄여주는 효과가 있다.
> **직교성이란?**
> * 공통점이 없는 기능들이 잘 분리되어 있다.
> * 기능을 원자적으로 쪼개 제공한다.
> * MSA아키텍쳐는 직교성이 높고, 모놀리식 아키텍쳐는 직교성이 낮다.
* ex. List의 subList()와 indexOf()
	* 지정된 범위의 부분 리스트에서의 인덱스를 찾는 기능을 두 메서드로 구현할 수 있다.

2. 매개변수 여러개를 묶어주는 도우미 클래스 만들기
* 도우미 클래스는 정적 멤버 클래스로 만든다.
* ex. 트럼프 카드 객체를 만들 때 숫자와 도형객체를 생성하여 만든다고 할때, 이 두개를 묶어주는 도우미 클래스를 만들어 하나의 매개변수로 주고받을 수 있다.

3. 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용하기
* 빌더 패턴과 도우미 클래스를 함께 사용하기
* 매개변수 타입으로는 클래스보다 인터페이스로 받자.
	* hashmap 대신 map 을 사용하자.
	* 만약 입력 데이터가 다른 형태로 존재하면 복사비용을 따로 치뤄야한다.
* boolean보다는 원소2개짜리 열거타입이 낫다.
	* 가독성이 올라가고 나중에 유지보수할 때 선택지를 추가하기 쉽다.
	* 열거 타입에 대한 의존성을 부여할 수 있다. (Item 34)


<!--

```java

```
 -->
