# Item 63 - 문자열 연결은 느리니 주의하기

#### 문자열을 + 로 합치기
```java
public String statement() {
	String result = "";
	for (int i = 0; i < numItems(); i++)
		result += lineForItem(i); // 문자열 연결 return result;
}
```
* 문자열 연결 연산자로 문자열 n개르 잇는 시간은 n^2에 비례하는 시간이 걸린다.
* String은 불변 객체이기 때문에 새로운 객체를 만들어 더하는 과정이 필요하다.

#### StringBuilder 사용하기
```java
public String statement2() {
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH); 
	for (int i = 0; i < numItems(); i++)
		b.append(lineForItem(i)); 
	return b.toString();
}
```
* 해당 방법이 훨씬 빠르다.


**정리**
* 많은 문자열을 연결할 때는 + 연산을 쓰지 말자.
	* 단일 스레드 환경에서는 StringBuilder가 더 빠르다.
	* 멀티 스레드 환경에서는 StringBuffer가 더 빠르다.
<!--
```java

```
 -->