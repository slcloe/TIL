# Item 84 - 프로그램의 동작을 스레드 스케줄러에 기대지 말기

#### 정확성이나 성능이 스레드 스케쥴러에 따라 달라지는 프로그램
**=> 다른 플랫폼에 이식하기 어렵다. => 이식성이 나쁘다.**

#### 이식성이 좋은 프로그램 작성법
* 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 많아지지 않도록 하는 것이다.
	* 스레드는 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.
	* 스레드는 절대 바쁜 대기(busy waiting) 상태가 되면 안 된다.
```java
public class SlowCountDownLatch {
	private int count;
	public SlowCountDownLatch(int count) { 
		if (count < 0)
			throw new IllegalArgumentException(count + " < 0"); 
		this.count = count;
	}

	public void await() {
		while (true) { 
			synchronized(this) {
				if (count == 0) 
					return;
			} 
		}
	}

	public synchronized void countDown() { 
		if (count != 0)
			count--; 
	}
}
```
> Thread.yield는 테스트 할 수 없기 때문에 사용하지 말자.
  
* 실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 실행되도록 만들어야한다.
	* 스레드 우선순위를 조절할 수 있지만, 이식성이 가장 나쁘기 때문에 사용하지 말자.


<!--
```java

```
 -->