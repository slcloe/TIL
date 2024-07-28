# Item 87 - 커스텀 직렬화 형태를 고려해보기

#### 기본 직렬화
* 객체가 포함한 모든 데이터 + 객체를 통해 접근할 수 있는 모든 객체 + topology
* 객체의 물리적 표현 == 논리적 내용 이라면 기본 직렬화로 구현해도 괜찮다.
##### 기본 직렬화 형태에 적합한 예시
```java
public class Name implements Serializable {
/**
* 성. null이 아니어야 함. * @serial
*/
private final String lastName;
/**
* 이름. null이 아니어야 함. * @serial
*/
private final String firstName;
/**
* 중간이름. 중간이름이 없다면 null. * @serial
*/
private final String middleName;
... // 나머지 코드는 생략 
}

```

##### 기본 직렬화 형태에 적합하지 않은 예시
```java
public final class StringList implements Serializable { private int size = 0;
private Entry head = null;
private static class Entry implements Serializable { String data;
Entry next;
Entry previous; }
... // 나머지 코드는 생략 }

```
* 물리적 : 문자열들을 이중 연결 리스트로 연결
* 논리적 : 문자열을 표현함

#### 기본 직렬화 형태의 단점
1. 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
	* 이후 수정된 부분일지라도 관련 코드를 제거할 수 없다.
2. 너무 많은 공간을 차지할 수 있다.
3. 시간이 너무 많이 걸릴 수 있다.
	* topology 에 대한 정보가 없어 그래프를 직접 순회해야한다.
4. 스택 오버플로를 일으킬 수 있다.
	* 객체 그래프를 재귀 순회할 때, 직렬화의 사이즈가 크면 에러가 난다.


#### 커스컴 직렬화
* 논리적인 구성만 직렬화 구성에 포함한다.
* transient 한정자를 통해 해당 인스턴스 필드를 기본 직렬화에서 제외 시킨다.
	* 역직렬화 시 기본값으로 초기화된다.
##### 커스텀 직렬화 예시
```java
public final class StringList implements Serializable { private transient int size = 0;
private transient Entry head = null;
// 이제는 직렬화되지 않는다. private static class Entry {
String data; Entry next; Entry previous;
}
// 지정한 문자열을 이 리스트에 추가한다.
public final void add(String s) { ... }
/**
* 이 {@code StringList} 인스턴스를 직렬화한다. *
* @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
* ({@code int}), 이어서 모든 원소를(각각은 {@code String})
* 순서대로 기록한다. */
private void writeObject(ObjectOutputStream s) throws IOException {
s.defaultWriteObject(); s.writeInt(size);
// 모든 원소를 올바른 순서로 기록한다.
for (Entry e = head; e != null; e = e.next)
s.writeObject(e.data); }
private void readObject(ObjectInputStream s)
throws IOException, ClassNotFoundException { s.defaultReadObject();
int numElements = s.readInt();
// 모든 원소를 읽어 이 리스트에 삽입한다.
for (int i = 0; i < numElements; i++)
add((String) s.readObject()); }
... // 나머지 코드는 생략 }

```
* 동작 과정
	1. transient 필드 도착
	2. defaultWriteObject, defaultReadObject 호출
	3. writeObject, ReadObject 호출


#### transient 한정자를 추가할 때
1. 캐시된 해시 값처럼 다른 필드에서 유도되는 필드
2. JVM을 실행할 때마다 값이 달라지는 필드
3. 네이티브 자료구조를 가리키는 long 필드

#### transient 한정자를 생략할 때
* 해당 객체의 논리적 상태와 무관한 필드일 때

#### 동기화 메커니즘 적용
* 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에 적용한다.

```java
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
	s.defaultWriteObject();
}
```
* synchronized로 선언하여 thread-safe 객체에 기본 직렬화를 적용할 때
	* writeObject 도 synchronized 선언을 해야한다.
	* writeObject 메서드 안에서 동기화하고 싶다면 다른 부분과 락 순서를 똑같이 해야한다. (데드락 주의)

#### serialVersionUID 를 명시적으로 선언하자
* 성능 개선
	* 런타입 시 값 생성에 복잡한 연산을 수행하기 때문에 먼저 선언하면 이 오버헤드를 줄일 수 있다.
* 초기값 부여시 생기는 문제점 해소
* 만약 신버전으로 넘어갈 때 serialVersionUID를 수정하면?
	* 구버전과의 호환이 끊기게 된다.
<!--
```java

```
 -->