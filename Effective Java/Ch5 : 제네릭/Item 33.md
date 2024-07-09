# Item 33 - 타입 안전 이종 컨테이너 고려하기

#### 타입 안전 이종 컨테이너란?
* map, set에서 제네릭 타입 시스템의 값의 타입이 키로 쓰이는 것
* 타입토큰 : 컴파일 시 타입 정보와 런타입 시 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴

```java
static class Favorites {
    private final Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```
* key가 와일드 카드다.
* key-value사이의 타입 관계를 보증하지 않는다.
* type.cast는 형변환 연산자의 동적버전이다.
	* 주어진 인수가 class객체가 알려주는 타입과 동일한지 검사한다.
		* 같다면 인수를 그대로 반환
		* 다르다면 ClassCastException

```java
public void test() {
    Favorites f = new Favorites();

    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorites.class);

    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);

    System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
}

// 출력 결과 : java cafebabe Favorites
``` 

#### 타입 안전 이종 컨테이너의 제약

1. 악의적인 클라이언트가 class 객체를 제네릭이 아니라 타입으로 넘기면 타입 안정성이 깨진다.
```java
f.putFavorite((Class) Integer.class, "String");
``` 
  * 해당 코드를 그냥 실행하면 ClassCastException 을 던진다.

```java
public <T> void putFavorite(Class<T> type, T instance) {
	favorites.put(Objects.requireNonNull(type), type.cast(instance)); 
}
```
* 동적 형변환으로 런타입 타입 안전성을 확보할 수 있다.


2. 실체화 불가 타입에는 사용할 수 없다.
  * `List<String>.class` 와 같이 실체화가 되지 않으면 사용할 수 없다.

**슈퍼 타입 토큰**
* 위 제약을 해결하기 위해 슈퍼 타입 토큰을 사용하려는 시도가 있었다.
* 스프링에서는 ParameterizedTypeReference 라는 클래스로 미리 구현해놨다.

```java
f.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = f.getFavorite(new TypeRef<List<String>>(){});
``` 
* 슈퍼 타입 토큰 예시

##### 한정적 타입 토큰
* 위 예시에서 사용하는 타입 토큰은 기본적으로 비한정적이다.
* 한정적 타입 매개변수`<E extends Number>`나 한정적 와일드 카드`List<? extends Number>`를 이용하여 표현 가능한 타입을 제한하는 것을 한정적 타입 토큰이라고 한다.
* 에너테이션 API에서 적극 활용한다.
* 에너테이션 API는 대상 요소의 에너테이션을 런타임 시 읽어오는 기능을 한다.
```java
public <T extends Annotation>
  T getAnnotation(Class<T> annotationType);
```
* annotationType은 에너테이션 타입을 뜻하는 한정적 타입 토큰이다.
* 토큰으로 명시한 타입의 에너테이션이 해당 요소에 
	* 있다면 에너테이션을 반환하고 
	* 없으면 NULL을 반환한다.
=> 에너테이션이된 요소는 그 키가 에너테이션 타입인 타입 안전 이종 컨테이너다.

##### 한정적 타입 토큰을 받는 메서드에 Class<?> 타입의 객체를 넘기는 법
* `Class<? extends Annotation>`으로 형변환 할 수 있지만, 이 형변환은 비검사이기때문에 컴파일 경고가 뜬다.
* 해결책으로, asSubClass 메서드를 활용하자
	* 호출된 인스턴스 자신의 class객체를 인수가 명시한 클래스로 형변환한다.
	* 형변환에 성공하면 인수로 받은 클래스 객체를 반환한다.
	* 형변환에 실퍠하면 ClassCastException 을 던진다.
```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
  Class<?> annotationType = null; // 비 한정적 타입 토큰
  try {
    annotationType = Class.forName(annotationTypeName);
  } catch (Exception ex) {
    throw new IllegalArgumentException(ex);
  }

  return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
``` 

<!-- 
```java

``` 
-->