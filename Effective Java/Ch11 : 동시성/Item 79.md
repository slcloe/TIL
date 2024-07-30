# Item 79 - 과도한 동기화는 피하기

#### 과도한 동기화의 문제점.
1. 성능 저하
2. 교착상태 우려
3. 예측할 수 없는 동작 과정 ( 데이터 훼손 우려. )
* => 따라서, 동기화는 제어권을 절대로 클라이언트에 양도하며 안된다.
	1. 제정의 할 수 있는 메서드는 호출하면 안된다.
	2. 클라이언트가 넘겨준 함수 객체를 호출하면 안된다.

#### 잘못된 예 - 동기화 블록 안에서 alien method를 호출한다.
* alien method란?
	* 동기화된 영역을 포함한 클래스를 제외한 나머지 부분
```java
public class ObservableSet<E> extends ForwardingSet<E> {
	public ObservableSet(Set<E> set) { super(set); }
	private final List<SetObserver<E>> observers = new ArrayList<>();

	public void addObserver(SetObserver<E> observer) { 
		synchronized(observers) {
			observers.add(observer);
		} 
	}

	public boolean removeObserver(SetObserver<E> observer) { 
		synchronized(observers) {
			return observers.remove(observer); 
		}
	}

	private void notifyElementAdded(E element) {
		synchronized(observers) {
			for (SetObserver<E> observer : observers) 
				observer.added(this, element);
		}
	}

	@Override public boolean add(E element) { 
		boolean added = super.add(element); 
		if (added)
			notifyElementAdded(element); 
		return added;
	}

	@Override public boolean addAll(Collection<? extends E> c) { 
		boolean result = false;
		for (E element : c)
			result |= add(element); // notifyElementAdded를 호출한다. 
			return result;
	} 
}

@FunctionalInterface public interface SetObserver<E> {
	// ObservableSet에 원소가 더해지면 호출된다.
	void added(ObservableSet<E> set, E element); 
}

public static void main(String[] args) { 
	ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

	set.addObserver((s, e) -> System.out.println(e));

	for (int i = 0; i < 100; i++) 
		set.add(i);
}
```
* Observer 패턴에서 addObserver 과 removeObserver 메서드를 호출해 구독을 신청/해지 하는 코드이다.
* 0 부터 99를 출력한다.
```java
set.addObserver(new SetObserver<>() {
	public void added(ObservableSet<Integer> s, Integer e) {
		System.out.println(e);
		if (e == 23) 
			s.removeObserver(this);
	} 
});
```
* 값이 23이면 자기 자신을 제거하는 코드이다.
**결과**
* 23까지 출력한 다음 ConcurrentModificationException을 던진다.
	* added 메서드 호출과 notifyElementAdded의 호출 시점이 같기 때문이다.
* added() -> ObservableSet.removeObserver() -> observers.remove() 호출
	* 리스트에서 원소를 제거함과 동시에 리스트를 순회하고 있다.
	* notifyElementAdded() 에서 수행하는 순회는 동기화 블록 안에 있으므로 동시수정이 일어나지 않도록 보장하지만,
		* 자기 자신이 콜백을 거쳐 되돌아와 수정하는 것은 막지 못한다.

#### 잘못된 예 - 불필요한 백그라운드 스레드 사용하는 Observer
```java
set.addObserver(new SetObserver<>() {
	public void added(ObservableSet<Integer> s, Integer e) {
		System.out.println(e); if (e == 23) {
			ExecutorService exec = Executors.newSingleThreadExecutor(); 
			try {
				exec.submit(() -> s.removeObserver(this)).get();
			} catch (ExecutionException | InterruptedException ex) {
				throw new AssertionError(ex); 
			} finally {
				exec.shutdown(); 
			}
		} 
	}
});
```
* 이 코드를 수행하면 교착상태가 발생한다.
* `백그라운드 스레드`에서 s.removeObserver을 호출하면 observer를 제거하려고 하지만 `메인 스레드`가 락을 쥐고 있다.
* `메인 스레드`는 `백그라운드 스레드`가 관찰자를 제거하기만을 기다린다.
* => `점유와 대기`로 인한 교착상태 발생!

#### 해결책
1. alien method를 동기화 블록 바깥으로 옮긴다.
```java
private void notifyElementAdded(E element) { 
	List<SetObserver<E>> snapshot = null; 
	synchronized(observers) {
		snapshot = new ArrayList<>(observers); 
	}
	for (SetObserver<E> observer : snapshot) 
		observer.added(this, element);
}
```
* 관찰자 리스트를 복사해 쓰면 락 없이도 안전하게 순회할 수 있다.
* 열린 호출(open call) : 동기화 영역 바깥에서 호출되는 alien method call
	* alien method의 지속 시간을 알 수 없기 때문에, 동작 중에는 다른 스레드가 무한 대기 상태에 놓이게 된다.
	* 따라서 열린 호출을 통해 실패 방지와 동시성 효율을 개선할 수 있다.
  
2. CopyOnWriteArrayList 사용
	* 자바의 동시성 컬렉션 라이브러리에 있는 함수이다.
	* 내부를 변경하는 작업은 복사본을 만들어 수행하도록 구현되어 있다.
	* 따라서, 순회 시 락이 필요 없어 매우 빠르다.
```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();
public void addObserver(SetObserver<E> observer) {
	observers.add(observer); 
}
public boolean removeObserver(SetObserver<E> observer) {
	return observers.remove(observer); 
}
private void notifyElementAdded(E element) { 
	for (SetObserver<E> observer : observers)
		observer.added(this, element); 
}
```
* 명시적으로 동기화한 곳이 사라졌다.

3. **동기화 영역에서는 가능한 한 일을 적게하기**


#### 동기화의 성능
* 성능 주요 요소 : 
	1. 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간
	2. 가상머신의 코드 최적화 제한
* 가변 클래스 작성시 유의 사항
	1. 동기화를 전혀 하지 말고, 가변 클래스를 동시에 사용하는 클래스가 외부에서 동기화를 하기.
	2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들기
		1. 락 분할(lock splitting)
		2. 락 스트라이핑(lock striping)
		3. 비차단 동시성 제어(nonblocking concurrency control)
	3. 쓰레드로부터 안전하지 않다면 문서화를 하자

 




<!--
```java

```
 -->