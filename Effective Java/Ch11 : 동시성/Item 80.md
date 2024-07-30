# Item 80 - 스레드보다는 실행자, 태스크, 스트림을 애용하기

#### ExecutorService의 주요 기능
```java
ExecutorService exec = Executors.newSingleThreadExecutor();
exec.execute(runnable);
exec.shutdown();
```
1. 특정 태스크가 완료되기를 기다린다
2. 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invoke All 메서드)가 완료되기를 기다린다.
3. 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드).
4. 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용).
5. 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThread PoolExecutor 이용).


#### 실행자 서비스의 생성
* 작업 큐를 두개 이상의 스레드에서 처리하고 싶다면 다른 종류의 실행자 서비스를 생성해도 된다.
1. java.util.concurrent.Executors의 정적 패터리 사용
	* 보통 이 방법을 많이 쓴다.
2. ThreadPoolExecutor 클래스 사용
	* 스레드 풀 동작을 결정하는 속성에 대한 설정이 가능하다.
3. Executors.newCachedThreadPool
	* 작은 프로그램 혹은 서버일 때 사용
	* 하지만, 볼륨이 크다면 좋지 않다.
		* 요청받은 서비스가 즉시 스레드에서 실행된다. ( 큐를 거치지 않는다.)
		* 만약 가용할 스레드가 없다면 새로운 스레드를 생성한다.
		* 결국 CPU이용률이 100%까지 올라가게 된다.
4. Executors.newFixedThreadPool & ThreadPoolExecutor
	* 무거운 프로그램일 때 사용
	* Executors.newFixedThreadPool : 스레드 개수 고정
	* ThreadPoolExecutor : 스레드 통제 가능

#### 실행자 프레임워크
* 실행자 프레임워크 = 태스크(작업 단위) + 실행 메커니즘(실행자 서비스)
* 테스크는 Runnable, Callable의 종류로 이루어져 있다.
* ForkJoinTask
	* 포크-조인 풀이라는 특별한 실행자 서비스가 실행한다.
	* ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉜다.
	* ForkJoinPool를 구성하는 스레드들이 이 테스크를 처리한다.
		* 일을 먼저 끝낸 스레드는 다른 스래드의 남은 테스크를 추가로 처리한다.
		* 병렬 스트림이 ForkJoinPool로 구현되어 있다.









<!--
```java

```
-->
