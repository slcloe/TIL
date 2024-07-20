# Item 77 - 예외를 무시하지 말기


#### catch 블록을 비워두지 말자!
```java
try {
...
} catch (SomeException e) { 

}
```
* 위 코드 같이 catch 블록을 비워두면 예외를 분기한 의미가 없다!
* 예외를 무시하지 않고 바깥으로 전파시켜도 최소한의 디버깅 정보를 남긴 채 프로그램이 신속히 중단되게 할 수 있다.


#### 예외를 무시해도 될 때
```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // 기본값. 어떤 지도라도 이 값이면 충분하다. 
try {
	numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
	// 기본값을 사용한다(색상 수를 최소화하면 좋지만, 필수는 아니다). 
}
```
* 예로 FileInputStream 을 닫을 때
* 대신, 예외를 무시하겠다면
	1. 무시하기로 결정한 이유를 주석으로 남기기
	2. 예외 변수의 이름을 ignored로 바꾸기


<!--
```java

```
 -->