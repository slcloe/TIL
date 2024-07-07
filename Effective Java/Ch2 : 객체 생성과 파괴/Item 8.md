# Item 8 - finalizer 와 cleaner 사용 피하기

> ##### 자바의 객체 소멸자
> 1. finalizer
> 2. cleaner  


#### 객체 소멸자의 문제점
1. 백그라운드에서 실행, GC 통제 하에 있다.
2. 즉시 수행된다는 보장이 없다.
	=> 따라서, 제때 실행되어야 하는 작업은 절대 할 수 없다.
3. 동작 중 발생한 예외는 무시되거나 잔여 작업이 있더라도 종료된다.
4. GC 효율을 떨어뜨리기 때문에 성능이 약 5배정도 느려진다.
5. finalizer 사용한 클래스는 finalizer 공격에 노출되어 보안문제가 발생할 우려가 있다.
> **공격 원리**
> 1. 생성자 혹은 직렬화 과정에서 예외 발생 
> 2. finalizer 수행
> 3. 정적 필드에 자신의 참조를 할당하기 때문에 GC가 수집할 수 없는 메모리가 됨.
> 4. 미완성객체의 메서드를 호출하게 되면 어떤 에러가 발생할지 모른다.

> **해결책**  
> 아무일도 하지 않는 finalize 메서드를 만들고 final로 선언하여 예방하기
> =>  `final 클래스는 하위 클래스를 만들 수 없으니 해당 공격에서 안전함.`

#### 객체소멸자의 대체제

* AutoCloseable 구현하기
close 메서드를 사용 -> 객체의 유효성 false를 필드에 기록  
-> 만약 객체가 다시 불렸다면 throw IllegalStateException  

#### 객체소멸자의 적절한 사용

1. 안전망의 역할의 finalizer 제공  
	ex) FileInputStream, FileOutputStream, ThreadPollExecutor  
2. 네이티브 피어와 연결된 객체에서의 사용
	###### 네이티브 피어란?
	> 일반 자바 객체가 네이티브 메서드를 통해 기능을 위힘한 네이티브 객체

	GC는 네이티브 피어를 회수할 수 없다.  
	=> 객체소멸자가 메모리를 회수한다.  
	* 하지만 성능저하를 감당해야하고 즉시 회수해야한다면 close메서드 사용.

3. AutoCloseable 과 try-with-resources 사용
```java
// Room 구현체 일부
@Override public void close() {
  cleanable.clean(); 
}

// 바른 예시
public class Adult {
  public static void main(String[] args) {
    // Room 객체는 AutoCloseable 구현이 된 상태이다.
    try (Room myRoom = new Room(7)) { 
	  System.out.println("done"); 
    }
  } 
}
```

```java
// 바르지 않은 예시
public class Teenager {
  public static void main(String[] args) {
    new Room(99);
    System.out.println("done"); 
  }
}
```
=> System.exit을 호출할 때 cleaner 동작 여부는 보장하지 않는다.


















