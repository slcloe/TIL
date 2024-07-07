# Item 7 - 다 쓴 객체 참조를 해제하기

메모리 누수를 막기 위한 방안
1) 해당 참조를 다 썼을 때 null 처리하기
2) 캐시의 사이클 확인하기
3) 리스너, 콜백 주의하기

#### 1. 해당 참조를 다 썼을 때 null 처리하기
```java
// Stack 의 pop method
public Object pop() {
	if (size == 0) throw new EmptyStackException();
	return elements[--size];
}
```

* `elements[--size]` 부분에서 메모리 누수가 일어난다.
	GC는 객체가 참조하는 모든 객체를 회수해가지 못하기 때문

```java
// Stack 의 pop method (수정본)
public Object pop() {
	if (size == 0) throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] == null; // 다 쓴 객체 참조 해제
	return result;
}
```
* 주의 ) 객체 참조를 null 처리하는 일은 예외적인 경우다.!
* 자기 메모리를 직접 관리하는 클래스인 경우 메모리 누수를 조심하자!



#### 2. 캐시의 사이클 확인하기
객체 참조를 캐시에 넣고 까먹었다면?
* WeakHashMap을 사용해 캐시만들기
	=> 다 쓴 엔트리는 즉시 자동으로 제거됨.
* 쓰지 않는 엔트리의 가치를 떨어뜨리는 방식 사용
	1. ScheduledThreadPollExecutor같은 백그라운드 스레드 활용
	2. 캐시에 새 엔트리를 추가할 때, 오래된 엔트리 삭제
		ex) LinkedHashMap의 removeEldestEntry
	3. java.lang.ref 패키지 활용

#### 3. 리스너, 콜백 주의하기
* 클라이언트가 콜백 등록 후 해지 하지 않았을 때 콜백은 계속 쌓이게 됨.
* 콜백을 weak reference로 저장하면 GC가 알아서 수거해간다.
	ex)  WeakHashMap사용