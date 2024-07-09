# Item 20 - 추상 클래스보다는 인터페이스로 구현하기

#### 추상클래스와 인터페이스 특징

##### 공통점
* 인스턴스 메서드를 구현 형태로 제공할 수 있다. (java 8 이후)

##### 차이점
* 추상클래스 : 단일 상속, 상속 시 새로운 타입으로 취급
* 인터페이스 : 다중 상속, 상속 시 같은 타입으로 취급

### 인터페이스의 장점
#### 1. 믹스인(mixin)
* 믹스인 : 클래스가 구현할 수 있는 타입, 특정 선택적 행위를 제공한다고 선언
 ex ) Comparable, Iterable, AutoCloseable
	* 추상클래스는 단일 상속만 지원하기 때문에 믹스인 삽입이 불가능하다.

#### 2. 계층구조가 없는 타입 프레임워크 만들기

```java
public interface Singer {
	void sing(Song s);
}

interface Songwriter {
	void compose(int chartPosition);
}

interface SingerSongWriter extends Singer, Songwriter {
	void strum();
	void actSensitive();
}
```
* 해당 구조를 만약 클래스로 정의한다면, 2^n개로 구성해야한다.


#### 3. 래퍼클래스 사용하기
* 래퍼클래스 : 기존에 인터페이스를 구현한 클래스를 주입받아 기존 구현체에 부가기능을 손쉽게 추가할 수 있는 클래스
	* 이를 데크레이터 패턴이라고 한다.

##### 4. 디폴트 메서드
* 인터페이스 메서드 중 구현 방법이 명확하다면 디폴트 메서드를 사용할 수 있다.
	* 단, Object클래스에서 제공하는 메서드는 불가하다. ex. equals(), hashCode()
	* public 이 아닌 정적맴버도 가질 수 없다. (private 정적메서드는 예외)
	* 내가 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

#### 5. 추상 골격 구현 클래스 활용
* 인터페이스와 추상 골격 구현 클래스를 함께 사용하면 인터페이스와 추상 클래스의 장점을 모두 취할 수 있다.
	* 인터페이스를 통해 타입 정의, 디폴트 메서드 정의
	* 골격 구현 클래스는 나머지 메서드 정의
	* 상속해서 사용하는 것을 가장하므로 상속에 대한 설계, 문서화 지침을 꼭 따라야한다.
* 골격 구현 작성
	1. 인터페이스에서 다른 메서드들의 구현에서 사용되는 기반 메서드 선정
	2. 기반메서드 = 골격 구현 추상 메서드
	3. 기반메서드로 구현할 수 있는 메서드는 디폴트 메서드로 구현

=> 이는 `템플릿 메서드 패턴`이라고 불린다.
관례상, 이름이 Interface라면, 골격 구현 클래스의 이름은 AbstractInterface로 짓는다.
ex. AbstractCollection, AbstractSet, AbstractList...

```java
static List<Integer> intArrayAsList(int[] a) {
  Objects.requireNonNull(a);

  return new AbstractList<>() {
    @Override
    public Integer get(int i) {
      return a[i]; // auto-boxing
    }

    @Override
    public Integer set(int i, Integer val) {
      int oldVal = a[i];
      a[i] = val; // auto-unboxing
      return oldVal; // auto-boxing
    }

    @Override
    public int size() {
      return a.length;
    }
  }
}
```
* Adapter구조 : int 배열을 받아 Integer의 인스턴스의 리스트 형태로 보여줌
* 오토 박싱/언박싱 때문에 성능은 좋지 않다.
* 익명 클래스 사용

> **골격 클래스의 장점**
> * 추상 클래스처럼 구현을 도와준다.
> * 추상 클래스로 타입을 정의할 때 단일 상속의 한계를 극복할 수 있다.

> **시뮬레이트한 다중 상속**이란?
> * 인터페이스를 구현한 클래스에서 골격 구현을 확장한 private내부 클래스를 정의하고
> * 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 방식


