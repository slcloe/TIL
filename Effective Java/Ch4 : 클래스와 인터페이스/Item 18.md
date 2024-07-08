# Item 18 - 상속보다는 컴포지션을 사용하기

#### 클래스 상속의 단점

상속은 캡슐화를 깨트린다.
	* 상위 클래스에서 내부 구현이 변경되면 하위 클래스가 오작동할 수 있다.

#### 상속의 단점 예시

1. 자기사용
```java
static class InstrumentedHashSet<E> extends HashSet<E> {
   private int addCount = 0;

   public InstrumentedHashSet() {
   }

   public InstrumentedHashSet(int initCap, float loadFactor) {
       super(initCap, loadFactor);
   }

   @Override
   public boolean add(E e) {
       addCount++;
       return super.add(e);
   }

   @Override
   public boolean addAll(Collection<? extends E> c) {
       addCount += c.size();
       return super.addAll(c);
   }

   public int getAddCount() {
       return addCount;
   }
}

@Test
public void instrumentHashSetTest() {
   InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
   s.addAll(List.of("A", "B", "C"));
   System.out.println("s.getAddCount() = " + s.getAddCount());
}
```
* addCount의 수는 3이여야 하지만 실제로는 6이 출력된다.
* InstrumentedHashSet.addAll()에서 addCount + 3 -> HashSet.addAll() 호출
* HashSet.addAll()은 각 원소에 대해 add()메서드를 사용한다.
* InstrumentedHashSet.add() 실행 addCount +3
* 각 원소당 2씩 늘어났다.

=> 하위 클래스에서 addAll()을 재정의하지 않으면 문제를 해결 가능함.
* 하지만 HashSet.addAll()메서드가 add()메서드를 통해서 구현된다는 사실은 API문서에도 없음

자기사용
* 자신의 다른 부분을 사용하는 방식
* 다음 릴리즈에서 유지될지 알 수 없다.


2. 하위 클래스의 캡슐화가 깨져버리는 다른 경우들
 * 상위 클래스에서 보안상의 이유로 컬렉션에서 원소를 추가할 때 특정 제약조건이 생겼다면?
	- 원소를 추가하는 모든 메서드를 재정의하고 제약조건을 먼저 검사한다.
 * 하지만 상위 클래스에 새로운 원소 추가 메서드가 생긴다면?
	- 실제로 HashTable, Vector를 컬렉션F/W에 포함했을 때, 심각한 보안 이슈가 발생했다.

 * 상위 클래스에 추가된 메서드가 하필 하위 클래스에 추가한 메서드와 일치하고 반환타입만 다르다면?
	- 컴파일 에러 발생
	- 반환타입까지 같다면 메서드 재정의를 한 것과 같다.

#### 상속 시 유의점
1. is-a 관계일 때만 상속하기
	* ex1 ) Stack은 Vector를 상속받지 말았어야 했다.
	* ex2 ) Properties은 HashTable을 상속받지 말았어야 했다.
		-> Properties의 get(), getProperty()의 결과값이 다르다.
2. 상속하려는 클래스 API의 결험까지 상속된다.


#### 컴포지션

* 상속 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하는 것
* 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.
forwarding : 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환하는 것
forwarding method : 새 클래스의 메서드들

```java
// 전달 클래스
static class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> set) {
        this.s = set;
    }

    @Override public Spliterator<E> spliterator() {
        return s.spliterator();}

    @Override public int size() { return s.size();}

    @Override public boolean isEmpty() {return s.isEmpty();}

    @Override public boolean contains(Object o) {return s.contains(o);}

    @Override public Iterator<E> iterator() {return s.iterator();}

    @Override public Object[] toArray() {return s.toArray();}

    @Override public <T> T[] toArray(T[] a) {return s.toArray(a);}

    @Override public boolean add(E e) {return s.add(e);}

    @Override public boolean remove(Object o) {return s.remove(o);}

    @Override public boolean containsAll(Collection<?> c) {return s.containsAll(c);}

    @Override public boolean addAll(Collection<? extends E> c) {return s.addAll(c);}

    @Override public boolean retainAll(Collection<?> c) {return s.retainAll(c);}

    @Override public boolean removeAll(Collection<?> c) {return s.removeAll(c); }

    @Override
    public void clear() {
        s.clear();
    }
}

// 래퍼 클래스
static class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> set) {
        super(set);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```
* InstrumentedSet 은 ForwardingSet을 상속하여 만들어졌다.
* ForwardingSet : Set을 구현한 객체를 사용할 뿐, 기능을 재정의 하지 않는다.
* `데코레이터 패턴` => TreeSet, HashSet 등 Set 인터페이스만 구현하면 적용이 가능하다.
* `래퍼 클래스` :  다른 Set 인스턴스를 감싸고 있다. ex) InstrumentedSet
	* 래퍼 클래스는 단점이 거의 없다.
	* 하지만, 콜백 프레임워크랑 어울리지 않는다.
