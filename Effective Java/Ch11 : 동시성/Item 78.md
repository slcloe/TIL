# Item 78 - 공유 중인 가변 데이터는 동기화해 사용하기

#### 동기화
* 한 스레드가 변경하는 중(상태가 일관되지 않은 순간) 객체의 상태를 다른 스레드에서 접근하지 못하도록 하는 것.
##### Synchronized
* 해당 메서드나 블록을 단일 스레드에서 수행하도록 보장하는 키워드 이다.
* 동기화 상태에서는 일관성이 깨진 상태를 볼 수 없다.
	* = 락에 의해 이전의 수정 최종 결과를 보여준다.
* 언어 명세상 long 과 double외의 변수의 읽고 쓰는 동작은 atomic 하다.
> **자바에서의 스레드는?**
> * 스레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다고 보장한다.
> * 하지만, 스레드가 저장한 값이 다른 스레드에게 보이는가는 보장하지 않는다.
> * 따라서, 동기화는 안정적인 통신에 꼭 필요하다.

#### 쓰레드를 멈추는 올바른 방법
* Thread.stop 메서드는 사용시 데이터가 훼손될 수 있기 때문에 사용하지 말자.
* 동기화를 사용하여 쓰레드를 멈추자
##### 올바르지 않은 코드
```java
public class StopThread {
	private static boolean stopRequested;
	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> { 
			int i = 0;
			while (!stopRequested) 
				i++;
		}); 
		backgroundThread.start();

		TimeUnit.SECONDS.sleep(1);
		stopRequested = true; 
	}
}

```
* 위 코드에서는 동기화를 하지 않았기 때문에, 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 볼지 알 수 없다.
* 따라서 무한반복에 빠지게 된다.
* 동기화에 빠지면 가상머신이 아래와 같은 hoisting 최적화를 수행할 수 있다.
```java
// 원래 코드
while (!stopRequested)
	i++;
// 최적화한 코드
if (!stopRequested)	
	while (true) i++;
```
* 만약 이렇게 구현된다면 프로그램음 응답 불가 상태가 된다.

##### 해결 코드 - 쓰기, 읽기 메서드 모두 동기화하기
```java
public class StopThread {
	private static boolean stopRequested;

	private static synchronized void requestStop() {
		stopRequested = true; 
	}
	private static synchronized boolean stopRequested() {
		return stopRequested; 
	}

	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> { int i = 0;
		while (!stopRequested()) 
			i++;
		}); 
		backgroundThread.start();
		
		TimeUnit.SECONDS.sleep(1);
		requestStop(); 
	}
}
```
* requestStop, stopRequested 필드를 동기화해 접근했다.


##### 해결 코드 - volatile 필드를 사용하기
```java
public class StopThread {
	private static volatile boolean stopRequested;

	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> { 
			int i = 0;
			while (!stopRequested) 
				i++;
		}); 
		backgroundThread.start();

		TimeUnit.SECONDS.sleep(1);
		stopRequested = true; 
	}
}
```
* stopRequested 필드를 volatile로 선언했다.
* **volatile 한정자란?**
	* 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.
	* 동기화의 두 효과중 통신만 지원한다. (원자성까지는 보장 못함.)

##### 동기화가 필요할 경우
```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
	return nextSerialNumber++; 
}
```
* 이 메서드는 항상 고유한 값을 반환해야한다.
* 하지만 증가연산자(++) 때문에, nextSerialNumber 필드에 두 번 접근하게 되어 값의 원자성을 보장할 수 없다.
	1. 값을 읽을 때
	2. (++ 한) 새로운 값을 저장할 때
	* 스레드가 1과 2사이에 접근하게 될 경우 원래는 1이 증가한 값을 돌려받지 못하고 이전 상태의 값을 받게 된다. ( = 안전실패)

###### 해결방법
1. generateSerialNumber 메서드에 synchronized 한정자 붙이기
	* 동시에 호출해도 간섭하지 않는다.
	* 이때, volatile은 제거해야한다.
2. AtomicLong 사용
	* java.util.concurrent.atomic 이 패키지는 락 프리를 지원하는 클래스들이 담겨 있다.
	* 동기화의 효과인 통신과 원자성을 지원한다.
	```java
	private static final AtomicLong nextSerialNum = new AtomicLong();

	public static long generateSerialNumber() {
		return nextSerialNum.getAndIncrement(); 
	}
	```
3. 가변 데이터를 공유하지 않기
	* 불변 데이터만 공유하면 좋다.
	* 가변 데이터는 단일 쓰레드에서만 쓰자...!
4. 공유하는 부분만 동기화 하기
	* 이렇게 되면, 객체를 수정하지 않았다면 다른 스레드들과 동기화없이 값을 공유할 수 있다.
	* 이런 행위를 안전 발행이라고 한다.
	* ex1. 클래스 초기화 과정에서 정적필드, volatile필드, final 필드, 락을 통해 접근하는 필드 사용.
	* ex2. 동시성 컬렉션에 저장

<!--
```java

```
 -->