# Item 81 - wait와 notify보다는 동시성 유틸리티를 애용하기

#### java.util.concurrent의 동시성 유틸리티
* 3개의 범주로 나눌 수 있다.
1. 실행자 프레임워크 (Item 80)
2. 동시성 컬렉션
3. 동기화 장치


#### 동시성 컬렉션
* List, Queue, Map 같은 표준 컬렉션 인터페이스와 동시성을 결합하여 구현한 고성능 컬렉션
	* 동시성을 효과적으로 구현하기 위해 동기화를 각자의 내부에서 수행한다.
	* => 동시성 무력화는 불가능하고, 외부에 추가적인 락 사용은 성능 저하를 유발할 수 있다.

* 여러 메서드를 원자적으로 묶어 호출하는 것은 불가능하다.
	* => 상태 의존적 수정 메서드를 사용한다.
		> **상태 의존적 수정 메서드란?** 
		> 여러 기본 동작을 하나의 원자적 동작으로 묶는 것.

##### 예시 : BlockingQueue
* take() 를 사용하여 큐의 첫 원소를 꺼낸다.
	* 이때, 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다.
* BlockingQueue 는 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다. 
* ThreadPoolExecutor를 포함한 대부분의 실행자 서비스(Item 80) 구현체에서 이 BlockingQueue를 사용한다.

#### 동기화 장치
* 스레드가 다른 스레드를 기다리게 하여서 서로의 작업을 조율한다.
* 동기화 장치 예시
	* CountDownLatch, Semaphore, CyclicBarrier, Exchanger, Phaser

##### CountDownLatch
* 하나 이상의 스레드가 다른 스레드의 작업이 끝날때 가지 기다리게 한다.
* 구현 방법
	* 생성자로 int를 받으며, 이 값을 통해 countDown 메서드의 호출 횟수를 지정하여 대기 중인 스레드의 활성 여부를 결정한다.

```java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
	CountDownLatch ready = new CountDownLatch(concurrency); 
	CountDownLatch start = new CountDownLatch(1);
	CountDownLatch done = new CountDownLatch(concurrency);

	for (int i = 0; i < concurrency; i++) { 
		executor.execute(() -> {
			// 타이머에게 준비를 마쳤음을 알린다. 
			ready.countDown();
			try {
				// 모든 작업자 스레드가 준비될 때까지 기다린다. 
				start.await();
				action.run();
			} catch (InterruptedException e) { 
				Thread.currentThread().interrupt();
			} finally {
				// 타이머에게 작업을 마쳤음을 알린다.
				done.countDown(); 
			}
		}); 
	}

	ready.await(); // 모든 작업자가 준비될 때까지 기다린다. 
	long startNanos = System.nanoTime(); 
	start.countDown(); // 작업자들을 깨운다. 
	done.await(); // 모든 작업자가 일을 끝마치기를 기다린다. 
	return System.nanoTime() - startNanos;
}
```
* CountDownLatch를 3개 사용했다.
* ready : 작업자 스레드들의 준비 완료를 타이버 스레드에 통지
* start : countDown을 호출해 작업자 스레드들을 깨운다.
* down : 마지막으로 남은 작업자 스레드의 동작을 마치고 done.countDown을 호출
* 타이머 스레드 : 종료 시각을 기록

* 시간 간격을 측정할 때는 System.nanoTime을 사용하자. ( System.currentTimeMills보다)
	* 시스템의 실시간 시계의 시간 보장에 영향받지 않음.


#### 그럼에도 불구하고, wait, notify를 사용해야한다면
##### wait
* wait메서드는 스레드 조건이 충족되기를 기다릴때 사용한다.
* 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.
```java
synchronized (obj) {
	while (<조건이 충족되지 않았다>)
		obj.wait(); // (락을 놓고, 깨어나면 다시 잡는다.) 

	... // 조건이 충족됐을 때의 동작을 수행한다.
}

```
* wait 메서드를 사용할 때는 반드시 wait loop 관용구를 사용하자.
	* 반복문을 통해서 wait호출 전후로 조건을 검사한다.

##### notify
* 락 조건이 충족되지 않았는데 스레드가 동작하면 불변식이 깨질 수 있다.
	* **예시**
	* 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
	* 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify 를 호출한다.
	* 깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충 족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다.
	* 대기 중인 스레드가 (드물게) notify 없이도 깨어나는 경우가 있다. (spurious wakeup)
* 따라서 보통의 경우에는 notifyAll을 사용하자.
	* 모든 쓰레드가 깨어남을 보장 -> 항상 예상할 수 있음. -> 정확한 결과
	* 관련 없는 스레드가 실수로 혹은 악의 적으로 wait를 호출하는 공격으로부터 보호할 수 있다.
* 모든 스레드가 같은 조건을 기다리고, 조건이 충족될 때마다 단 하나의 스레드만 혜택을 받는다면
	* notify를 사용하자.



<!--
```java

```
 -->